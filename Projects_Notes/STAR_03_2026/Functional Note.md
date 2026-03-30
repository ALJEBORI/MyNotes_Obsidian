<font color="#e6ea74">SAE_SAE</font>

1. Données de production : 1 info trafic champ totaltrafic + 1 info lieux de la mesure sae_pr_n (l'emplacement physique de la station de comptage)

si vide   
2. complétion par référence (automatique) SI la section (dans le ref section) a une section de référence   
2.1 la section de référence est de type COMPTAGE, on récupère depuis sae_sae traficjour + sae_pr_n  
2.2 la section de référence est de type PEAGE, on récupère depuis pit trafic mais on met sae_pr_n à NULL

si vide  
3. mise dans le fichier à compléter manuellement => complétion par intégration de fichier. Mise en place de la valeur du trafic saisie manuellement et sae_pr_n = NULL

# ==**Note:**==
1.  We have peage (OD), comptage and toute réseau TR
2.  TR means either we will take peage or comptage dependeng on traffic representative (file from aprr)
3. We filter parcours iteneraire depending on referential (trafic representative) to get sections peage