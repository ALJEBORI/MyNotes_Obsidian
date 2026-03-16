
==**Joblet download _SHAREPOINT_FILES**==
Job Name: 
- This joblet read a list of files from sharepoint to local the file name is input to joblet from sharepoint_files_list

- input parameter  

| Input parameter           | data type | Comments                                                              |
| ------------------------- | --------- | --------------------------------------------------------------------- |
| sharepoint_ressource_path | string    |                                                                       |
| local_download_directory  | string    | where to put the downloaded files in local                            |
| sharepoint_file_filter    | string    | Regex file mask (file to be downloaded), ex   * *.csv, rapport_*.xlsx |
| is_archiving_enabled      | string    | is archiving needed true else false                                   |
| archive_folder_name       |           | name of archive folder                                                |
| azure_tenant_id           | string    |                                                                       |
| azure_client_id           | string    |                                                                       |
| azure_client_secret       | string    | context.parameter_filename                                            |
| drive_id                  | string    | context.STAR_LocalTmp_Path+ "/"                                       |

- First part create token



**Joblet de téléchargement (Download)**

- Ce joblet permet de télécharger des fichiers depuis un dossier SharePoint vers un répertoire local. Il prend en entrée les credentials Azure (depuis le contexte chiffré), le drive_id (depuis le contexte projet), le chemin du dossier SharePoint cible relatif à la racine du drive, le répertoire local de destination, ainsi qu'un filtre de fichiers au format glob (ex : *.csv, rapport_*.xlsx) — seuls les fichiers correspondant au masque seront téléchargés.
    
- Une option d'archivage automatique est disponible : lorsqu'elle est activée, chaque fichier téléchargé est déplacé dans un sous-dossier d'archive dans SharePoint. Ce dossier doit exister au préalable. En cas de doublon, le fichier est renommé automatiquement.
    

  

**Joblet d'upload (Upload)**

- Ce joblet permet d'envoyer l'ensemble des fichiers d'un répertoire local vers un dossier SharePoint. Il prend en entrée les credentials Azure (depuis le contexte chiffré), le drive_id (depuis le contexte projet), le répertoire local source, un filtre de fichiers au format glob (ex : *.xml) et le chemin du dossier de destination dans SharePoint relatif à la racine du drive.
    
- Les noms de fichiers contenant des espaces ou caractères spéciaux sont gérés automatiquement.







ist_items.name != null &&  list_items.microsoft_graph_downloadUrl != null &&  list_items.size > 0 && list_items.name.matches(
	((String)globalMap.get("sharepoint_file_filter"))
       	.replace(".", "\\.")
        .replace("*", ".*")
        .replace("?", ".")
)