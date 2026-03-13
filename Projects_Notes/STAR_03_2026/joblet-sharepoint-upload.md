# Joblet — SharePoint Upload via Microsoft Graph API

## Vue d'ensemble

Ce joblet permet d'**envoyer l'ensemble des fichiers d'un répertoire local vers un dossier SharePoint**, via l'API Microsoft Graph. Il itère automatiquement sur tous les fichiers du dossier correspondant au filtre fourni.

Il est conçu pour être réutilisable : un même job peut l'appeler plusieurs fois avec des configurations différentes (dossiers, filtres, drives distincts).

---

## Prérequis

Une application doit être enregistrée dans **Microsoft Entra ID** avec :

- Type d'authentification : **Application** (et non Delegated)
- Permission : **Sites.Selected** — pour cibler précisément le site SharePoint concerné
- Permission : **Files.ReadWrite.All** — pour écrire des fichiers dans SharePoint

---

## Variables de contexte

Ces variables remontent dans le job parent. Elles représentent les paramètres d'infrastructure stables et n'ont généralement pas besoin d'être modifiées.

| Variable | Valeur par défaut | Description |
|---|---|---|
| `MICROSOFT_GRAPH_BASEURL` | `https://graph.microsoft.com/v1.0` | URL de base de l'API Graph |
| `AZURE_AUTH_ENDPOINT` | `https://login.microsoftonline.com` | Endpoint d'authentification Azure |
| `AZURE_OAUTH_PATH` | `/oauth2/v2.0/token` | Chemin du token OAuth2 |
| `MICROSOFT_GRAPH_SCOPE` | `https://graph.microsoft.com/.default` | Scope Client Credentials |
| `MAX_NORMAL_UPLOAD_SIZE` | `250000000` (250 Mo) | Taille maximale d'un fichier uploadable |
| `MAX_UPLOAD_SESSION_CHUNK_SIZE` | `10485760` (10 Mo) | Taille des chunks pour upload sessionné |

---

## Paramètres d'entrée

Le joblet reçoit ses paramètres via un flux d'entrée (`tFixedFlowInput` ou équivalent).

### Credentials Azure

> 🔐 Ces 3 valeurs doivent impérativement provenir d'un **fichier de contexte chiffré** Talend. Ne jamais les saisir en clair.

| Colonne | Description |
|---|---|
| `azure_tenant_id` | GUID du tenant Azure AD |
| `azure_client_id` | App (client) ID de l'application Entra |
| `azure_client_secret` | Secret client de l'application Entra |

### Drive ID

> 📁 Doit provenir d'un **contexte projet**. Permet de cibler des drives différents dans un même job.

| Colonne | Description |
|---|---|
| `drive_id` | ID de la librairie de documents SharePoint cible |

### Paramètres fonctionnels

| Colonne | Type | Description |
|---|---|---|
| `local_directory_path` | String | Répertoire local contenant les fichiers à envoyer (ex : `C:/data/exports/`) |
| `file_filter` | String | Filtre de fichiers au format glob (ex : `*.csv`, `export_*.xml`) |
| `sharepoint_folder_path` | String | Chemin du dossier de destination dans SharePoint, relatif à la racine du drive (ex : `/Sortants/2025`) |

#### À propos du filtre de fichiers

Le filtre utilise la syntaxe glob :

| Filtre | Fichiers envoyés |
|---|---|
| `*.csv` | Tous les fichiers CSV |
| `export_*.xml` | Tous les fichiers XML commençant par `export_` |
| `*` | Tous les fichiers du répertoire |

> ℹ️ Le filtre est sensible à la casse sur les environnements Linux/Unix. Assurez-vous que la casse de vos noms de fichiers est cohérente avec le filtre configuré.

#### À propos de `sharepoint_folder_path`

Ce paramètre est relatif à la **racine du drive**. Le drive ID identifie déjà la librairie de documents ; le `sharepoint_folder_path` indique le dossier de destination à l'intérieur.

| Dossier cible | Valeur |
|---|---|
| Racine du drive | `/` |
| `Documents/Sortants` | `/Sortants` |
| `Documents/Exports/2025` | `/Exports/2025` |

#### Gestion des noms de fichiers

Les noms de fichiers contenant des **espaces ou caractères spéciaux** sont gérés automatiquement — ils sont encodés en URL avant l'envoi vers SharePoint.

#### Limite de taille

Les fichiers dépassant `MAX_NORMAL_UPLOAD_SIZE` (250 Mo par défaut) provoquent l'arrêt du job avec le code d'erreur `1`. Cette limite peut être ajustée dans les variables de contexte si nécessaire.

---

## Fonctionnement

```
INPUT (flux d'entrée)
  └─► Init           — Stockage des paramètres dans la globalMap
        └─► Auth     — Appel OAuth2 Client Credentials → Bearer token
              └─► [pour chaque fichier du répertoire local correspondant au filtre]
                    └─► Properties   — Lecture des métadonnées du fichier (taille, chemin, nom)
                          └─► Guard  — Vérification que le fichier ne dépasse pas 250 Mo
                                └─► Upload  — Envoi du fichier via PUT Graph API
```

---

## Codes d'erreur

| Code | Cause |
|---|---|
| `1` | Fichier dépassant la taille maximale autorisée (250 Mo) |
| `2` | Échec de l'upload d'un fichier |

---

## Exemple de configuration

```
Contexte chiffré :
  TENANT_ID     = <guid>
  CLIENT_ID     = <guid>
  CLIENT_SECRET = <secret>

Contexte projet :
  DRIVE_ID_SORTANTS = <drive-id>

tFixedFlowInput :
  azure_tenant_id      = context.TENANT_ID
  azure_client_id      = context.CLIENT_ID
  azure_client_secret  = context.CLIENT_SECRET
  drive_id             = context.DRIVE_ID_SORTANTS
  local_directory_path = "C:/data/exports/"
  file_filter          = "*.csv"
  sharepoint_folder_path = "/Sortants/2025"
```
