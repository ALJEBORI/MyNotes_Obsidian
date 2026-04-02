- <font color="#8db3e2", size="+3">Concept of PEAGE</font>
	   - OD (gare origine- gare destination)
	   - each OD has Iteniraire and each Iteniraire has sections
	   - OD characteristics is
		   1- Traffic
		   2- longure parcoures
		   3- Tarffic+longure parcoures = TMJ, IK
		-  Notations
			 1. GaG: Gare à gare  means Gare entrie = GareSorti like doing demi-tour autoroute or demi-tour A120, see fig below (A), <mark style="background:#fff88f">soustype=0 we have to include it in calculation IK, TMJ, table  fstg_soustypegare_s</mark>
			 2.  TLPC: Trajet le plus cher, when the vehicule treche ou dit j'ai pérdu le ticket  (pas de ticket entree), see fig below (B). <mark style="background:#fff88f">soustype >0 not included in the calculation, table fstg_soustypegare_s</mark> 
			 3. TLMC: Trajet le mois cher, just in case the information of the vehicle is not registered in the system due to   problem technique the APRR will take the responsibility of their error and apply TLMC, Also soutype of it is >0 <mark style="background:#fff88f">soustype >0 not included in the calculation, table fstg_soustypegare_s</mark>  see fig below (C)
			 4. Gare inconnu: when the ports of gare are forced to open like (jellet jeune) then at the destination Gare we don't know this vhicule enter from which . <mark style="background:#fff88f">soustype >0 not included in the calculation, table fstg_soustypegare_s</mark>  fig(D) below


Fig (A)     


![[Im_1_2.jpg]]




 - <font color="#8db3e2", size="+3">Concept of Comptage/ Peage</font>:      
      - Table SAE_SAE contains the information of all sections comptage and it is fille according to two ways:
          1.  Complétion Reference
          2. Compeletion manaul
          3. From refrence files comptage
    -  Table ODC_ODcommun contains sections peage like S1 (his trafic representative is P) or comptage (comptage because trafic representative has value T, even it is between two Gare) Like S2. All these data are in the table pit_parcoursitineraire
    - When calculating the trafic for section S1 it is peage and all his data in pit_parcoursitiniraires but for the section S2 we will search for his data in the table SAE_SAE because it is comptage . <mark style="background:#fff88f">important note the section S1 also available in SAE_SAE as there were cpt on it but we will not take it is data from SAE_SAE as it is of type P</mark>. 
    - There were some sections that are purly comptage (free no pay, autorout gratuite) but managed by APRR those are available only in SAE_SAE

		 
- <font color="#8db3e2", size="+3">Concept of Comptage</font>
  imagine you enter to Gare VSS  and out from Gare Vss soud , then between these two Gare there were many sections like S1, S2 and we put sensors on it (prn for counting). Then this is OD and contines section S1 like peage P and section S2 comptage T (this is specified in the traficrepresentative column)--> we will find in the table pit_parcoursitienirair  some sections peage P and some sections comptage T. And after Vss soud  you continue to Lyon where from Vss soud to Lyon there were only comptage sections (no entry gare or out Gare). Also note that there were some sections like S5 we don't have info about it so we will take his info like the section S4 (this done in the file ) completion reference if we don't find it there we will find it in the complétion manual (this two methods used to create the table SAE_SAE)


![[Im_3_4.jpg]]