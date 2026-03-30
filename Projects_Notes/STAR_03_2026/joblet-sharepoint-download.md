# Joblet — SharePoint Download via Microsoft Graph API

## Vue d'ensemble

Ce joblet permet de **télécharger des fichiers depuis un dossier SharePoint** vers un répertoire local, via l'API Microsoft Graph, avec une option d'archivage automatique des fichiers traités.

Il est conçu pour être réutilisable : un même job peut l'appeler plusieurs fois avec des configurations différentes (dossiers, filtres, drives distincts).

---

## Prérequis

Une application doit être enregistrée dans **Microsoft Entra ID** avec :

- Type d'authentification : **Application** (et non Delegated)
- Permission : **Sites.Selected** — pour cibler précisément le site SharePoint concerné
- Permission : **Files.Read.All** — pour lister et télécharger les fichiers
- Permission : **Files.ReadWrite.All** — si l'archivage est activé

---

## Variables de contexte

Ces 4 variables remontent dans le job parent. Elles représentent les paramètres d'infrastructure stables et n'ont généralement pas besoin d'être modifiées.

| Variable                  | Valeur par défaut                      |
| ------------------------- | -------------------------------------- |
| `MICROSOFT_GRAPH_BASEURL` | `https://graph.microsoft.com/v1.0`     |
| `AZURE_AUTH_ENDPOINT`     | `https://login.microsoftonline.com`    |
| `AZURE_OAUTH_PATH`        | `/oauth2/v2.0/token`                   |
| `MICROSOFT_GRAPH_SCOPE`   | `https://graph.microsoft.com/.default` |

---

## Paramètres d'entrée

Le joblet reçoit ses paramètres via un flux d'entrée (`tFixedFlowInput` ou équivalent).

### Credentials Azure

> 🔐 Ces 3 valeurs doivent impérativement provenir d'un **fichier de contexte chiffré** Talend. Ne jamais les saisir en clair.

| Colonne               | Description                            |
| --------------------- | -------------------------------------- |
| `azure_tenant_id`     | GUID du tenant Azure AD                |
| `azure_client_id`     | App (client) ID de l'application Entra |
| `azure_client_secret` | Secret client de l'application Entra   |

### Drive ID

> 📁 Doit provenir d'un **contexte projet**. Permet de cibler des drives différents dans un même job.

| Colonne | Description |
|---|---|
| `drive_id` | ID de la librairie de documents SharePoint cible |

### Paramètres fonctionnels

| Colonne                    | Type    | Description                                                                               |
| -------------------------- | ------- | ----------------------------------------------------------------------------------------- |
| `sharepoint_resource_path` | String  | Chemin du dossier cible relatif à la racine du drive (ex : `/Entrants`, `/Factures/2025`) |
| `local_download_directory` | String  | Répertoire local de destination                                                           |
| `sharepoint_file_filter`   | String  | Filtre de fichiers au format glob (ex : `*.csv`, `rapport_*.xlsx`)                        |
| `is_archiving_enabled`     | Boolean | Active l'archivage automatique après téléchargement                                       |
| `archive_folder_name`      | String  | Nom du sous-dossier d'archive dans SharePoint                                             |

#### À propos de `sharepoint_resource_path`

Ce paramètre est relatif à la **racine du drive**. Le drive ID identifie déjà la librairie de documents ; le `sharepoint_resource_path` indique où naviguer à l'intérieur.

| Dossier cible | Valeur |
|---|---|
| Racine du drive | `/` |
| `Documents/Entrants` | `/Entrants` |
| `Documents/Factures/2025` | `/Factures/2025` |

#### À propos du filtre de fichiers

Le filtre utilise la syntaxe glob, convertie automatiquement en expression régulière :

| Filtre | Fichiers récupérés |
|---|---|
| `*.csv` | Tous les fichiers CSV |
| `rapport_*.xlsx` | Tous les fichiers Excel commençant par `rapport_` |
| `export_202?_*.xml` | Fichiers XML avec un caractère quelconque à la place de `?` |

> ℹ️ Le filtre s'applique uniquement aux fichiers **directement dans le dossier ciblé**, sans récursivité dans les sous-dossiers. Les fichiers de taille 0 et les dossiers sont automatiquement exclus.

#### À propos de l'archivage

Lorsque `is_archiving_enabled = true`, chaque fichier téléchargé est déplacé dans le sous-dossier `archive_folder_name` dans SharePoint.

- Le dossier d'archive doit **exister au préalable** — le joblet ne le crée pas
- En cas de doublon, le fichier archivé est **renommé automatiquement**
- Si l'archivage est activé mais que le dossier est introuvable, le job s'arrête avec le code d'erreur `2` **avant tout téléchargement**

---

## Fonctionnement

```
INPUT (flux d'entrée)
  └─► Init          — Stockage des paramètres dans la globalMap
        └─► Auth    — Appel OAuth2 Client Credentials → Bearer token
              └─► List       — Listing des items du dossier SharePoint via Graph API
                    └─► Filtre     — Séparation : dossier d'archive / fichiers à télécharger
                          └─► [pour chaque fichier]
                                └─► Guard + Payload  — Vérification archive, préparation du PATCH
                                      └─► Download   — Téléchargement via @microsoft.graph.downloadUrl
                                            └─► Archive (si activé) — Déplacement via PATCH Graph API
```

---

## Codes d'erreur

| Code | Cause |
|---|---|
| `1` | Échec du listing des items SharePoint |
| `2` | Archivage activé mais dossier d'archive introuvable |
| `3` | Échec du téléchargement d'un fichier |
| `4` | Échec du déplacement vers le dossier d'archive |

---

## Exemple de configuration

```
Contexte chiffré :
  TENANT_ID     = <guid>
  CLIENT_ID     = <guid>
  CLIENT_SECRET = <secret>

Contexte projet :
  DRIVE_ID_ENTRANTS = <drive-id>

tFixedFlowInput :
  azure_tenant_id          = context.TENANT_ID
  azure_client_id          = context.CLIENT_ID
  azure_client_secret      = context.CLIENT_SECRET
  drive_id                 = context.DRIVE_ID_ENTRANTS
  sharepoint_resource_path = "/Entrants"
  local_download_directory = "C:/data/entrants/"
  sharepoint_file_filter   = "*.csv"
  is_archiving_enabled     = true
  archive_folder_name      = "Archive"
```



