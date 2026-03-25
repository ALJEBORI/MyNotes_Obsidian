
==**Joblet download _SHAREPOINT_FILES**==
Job Name: 
- This joblet read a list of files from sharepoint to local the file name is input to joblet from sharepoint_files_list

- input parameter  

| Input parameter           | data type | Comments                                                                                            |
| ------------------------- | --------- | --------------------------------------------------------------------------------------------------- |
| sharepoint_ressource_path | string    |                                                                                                     |
| local_download_directory  | string    | where to put the downloaded files in local    context.STAR_LocalTmp_Path+ "/"                       |
| sharepoint_file_filter    | string    | context.parameter_filename,   Regex file mask (file to be downloaded), ex   * *.csv, rapport_*.xlsx |
| is_archiving_enabled      | string    | is archiving needed true else false                                                                 |
| archive_folder_name       |           | name of archive folder                                                                              |
| azure_tenant_id           | string    | context.API_MS_GRAPH_SHAREPOINT_STAR_tenant_id                                                      |
| azure_client_id           | string    | context.API_MS_GRAPH_SHAREPOINT_STAR_client_id                                                      |
| azure_client_secret       | string    | context.API_MS_GRAPH_SHAREPOINT_STAR_client_secret                                                  |
| drive_id                  | string    |                                                                                                     |

 **==Sharepoint connection file:  CNX_DEV_API_MS_GRAPH_SHAREPOINT_STAR.properties==**

#Fri Mar 13 14:27:50 CET 2026
API_MS_GRAPH_SHAREPOINT_STAR_tenant_id
API_MS_GRAPH_SHAREPOINT_STAR_client_id
API_MS_GRAPH_SHAREPOINT_STAR_client_secret
API_MS_GRAPH_SHAREPOINT_STAR_host
API_MS_GRAPH_SHAREPOINT_STAR_site_name
API_MS_GRAPH_SHAREPOINT_STAR_resource







==**For drive_id according to the job use**==
STAR DRIVE_ID SHAREPOINT
SHAREPOINT_Coefficient_Repartition_DRIVE_ID
SHAREPOINT_SAE_COMPTAGE_DRIVE_ID
SHAREPOINT_KMP_Previsionnels_DRIVE_ID
SHAREPOINT_ReCalcul_Histo_DRIVE_ID
SHAREPOINT_Coefficient_Repartition_OD_DRIVE_ID
SHAREPOINT_Recette_Mensuelle_DRIVE_ID
SHAREPOINT_Calendrier_Reference_DRIVE_ID


**Joblet de téléchargement (Download)**

- Ce joblet permet de télécharger des fichiers depuis un dossier SharePoint vers un répertoire local. Il prend en entrée les credentials Azure (depuis le contexte chiffré), le drive_id (depuis le contexte projet), le chemin du dossier SharePoint cible relatif à la racine du drive, le répertoire local de destination, ainsi qu'un filtre de fichiers au format glob (ex : *.csv, rapport_*.xlsx) — seuls les fichiers correspondant au masque seront téléchargés.
    
- Une option d'archivage automatique est disponible : lorsqu'elle est activée, chaque fichier téléchargé est déplacé dans un sous-dossier d'archive dans SharePoint. Ce dossier doit exister au préalable. En cas de doublon, le fichier est renommé automatiquement.
    

  

**Joblet d'upload (Upload)**

- Ce joblet permet d'envoyer l'ensemble des fichiers d'un répertoire local vers un dossier SharePoint. Il prend en entrée les credentials Azure (depuis le contexte chiffré), le drive_id (depuis le contexte projet), le répertoire local source, un filtre de fichiers au format glob (ex : *.xml) et le chemin du dossier de destination dans SharePoint relatif à la racine du drive.
    
- Les noms de fichiers contenant des espaces ou caractères spéciaux sont gérés automatiquement.












==**FRK_DOWNLOAD_GRAPH_FILES  0.1 :**==

Input

Etap 1: Create token 
          Method: POST
          call  URL:  context.API_MS_GRAPH_SHAREPOINT_AZURE_AUTH_ENDPOINT
          Relative path: "/" + ((String)globalMap.get("azure_tenant_id")) + context.API_MS_GRAPH_SHAREPOINT_AZURE_OAUTH_PATH
  ![[Pasted image 20260316153623.png]]

Output JSON parse it get token save in Gvar:  "sharepoint_access_token"

Etap 2:  Liste files to download
             GET all files in the sharepoint_resource_path for that drive_id

 
             
Etap3: Extract data from returend json
           extract id, name,microsoft_graph_downloadUrl and size

Etap4: Filter the files using sharepoint_file_filter

ist_items.name != null &&  list_items.microsoft_graph_downloadUrl != null &&  list_items.size > 0 && list_items.name.matches(
	((String)globalMap.get("sharepoint_file_filter"))
       	.replace(".", "\\.")
        .replace("*", ".*")
        .replace("?", ".")
)


Etap5: Download the all files matche sharepoint_file_filter using tFileFetch

![[Pasted image 20260316154900.png]]



OK --> "Fichier '" +  globalMap.get("row10.name") + " téléchargé"  
             Code 0


"Echec du téléchargement du fichier '" + globalMap.get("row10.name") + "'"
context.Log_400_Code_File_Write_KO


==**FRK_UPLOAD_GRAPH_FILES  0.1**==

Input Parameter

| Input parameter        | data type | Comments                                                              |
| ---------------------- | --------- | --------------------------------------------------------------------- |
| local_folder_path      | string    |                                                                       |
| sharepoint_folder_path | string    | where to put the downloaded files in local                            |
| file_mask              | string    | Regex file mask (file to be downloaded), ex   * *.csv, rapport_*.xlsx |
| azure_tenant_id        | string    |                                                                       |
| azure_client_id        | string    |                                                                       |
| azure_client_secret    | string    | context.parameter_filename                                            |
| drive_id               | string    | context.STAR_LocalTmp_Path+ "/"                                       |


Etap 1: Create token   like joblet downlaod
Etap 2: 
    1- List all files int the local_folder_path with mask  file_mask (using tFileList_3)
    2-  Get the properties of the files using tFileProperties_3
    3-  Using tJavaRow_5 to preperare the paramers for upload call
          // --- 1️ Register file information from the input row ---
globalMap.put("file_path", input_row.abs_path);
globalMap.put("file_name", input_row.basename);

String rawPath = input_row.basename;
String encodedPath = java.net.URLEncoder.encode(rawPath, "UTF-8").replace("+", "%20").replace("%2F", "/");
globalMap.put("file_name_encoded", encodedPath);                               
//globalMap.put("file_name_encoded", URLEncoder.encode(input_row.basename, StandardCharsets.UTF_8).replace("+", "%20"));
globalMap.put("dir_name", input_row.dirname);
globalMap.put("total_file_length", input_row.size);

java.io.File file = new java.io.File(((String)globalMap.get("tFileList_3_CURRENT_FILEPATH")));
byte[] bytes = java.nio.file.Files.readAllBytes(file.toPath());

globalMap.put("current_file_bytes", bytes);
globalMap.put("current_file_length", Integer.toString(bytes.length));


  Etap 3: 
  PUT   context.API_MS_GRAPH_SHAREPOINT_MICROSOFT_GRAPH_BASEURL + "/drives/" + ((String)globalMap.get("drive_id")) + "/root:/" + ((String)globalMap.get("sharepoint_folder_path")) + ((String)globalMap.get("file_name_encoded")) + ":/content"


![[Pasted image 20260316162541.png]]


IF KO --> row14.message   Code: context.Log_300_Code_API_Post_KO



V 0.9: Remplacer le joblet FRK_Download_Sharepoint_Files par le nouveau joblet FRK_Download_Graph_Files.<br/>


