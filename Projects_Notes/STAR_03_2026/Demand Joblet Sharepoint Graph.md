Integration of new joblet sharepoint graph in the following flux  all are deployed in REC:

-<font color="#ffff00">STAR_SAE_LOAD_COMPLETION </font>

<font color="#ffff00">- STAR_ORDO_RECALCUL_HISTORIQUE </font>

<font color="#ffff00">- STAR_ORDO_KMP_PREVISIONNEL </font>

<font color="#ffff00">- STAR_INT_BRZ_REC_MENSUELLE </font>

<font color="#ffff00">- STAR_INT_BRZ_CALENDRIER_REF </font>

<font color="#ffff00">- STAR_ORDO_COEF_REPARTITION </font>

<font color="#ffff00">- STAR_SAE_COMPLETION</font>

<font color="#ffff00">- STAR_IHM_SANCT_DESANCT</font>

<font color="#ffff00">- STAR_ORDO_CALCUL</font>

<font color="#ffff00">- STAR_ORDO_CALCUL_2</font>

Flux That I updated and deployed on TMC :
<font color="#e6ea74">1. </font><font color="#e6ea74">STAR_ORDO_REC_MENSUELLE  TMC 0.1.25 </font>: Job ORDO, I  modified the child job 
      <font color="#e6ea74">STAR_INT_BRZ_REC_MENSUELLE</font>   <font color="#e6ea74">tmc 0.1.11</font>, the input file name is      
       **Recette mensuelle xlsxToCSV PointVirgule UTF-8**        
<font color="#e6ea74">2. STAR_ORDO_CALENDRIER_REF  TMC 0.1.32</font>: Job ORDO, I modified the child job   
       <font color="#e6ea74">  STAR_INT_BRZ_CALENDRIER_REF TMC 0.1.1</font> Input file name is 
         **Calendrier de reference APRR 2025.csv**
<font color="#ffff00"> 3. STAR_ORDO_COEF_REPARTITION  TMC  0.1.32</font>: Job ORDO, I modified the child job 
      <font color="#ffff00">STAR_INT_BRZ_COEF_REPARTITION_OD</font>, Input file name is 
    **Coeff_repartition APRR Juin 2025.txt**
 
