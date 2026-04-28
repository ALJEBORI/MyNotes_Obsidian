- change flux  STAR_ PREPARA_IK version to 0.1  Called by STAR_ORDO_CALCUL_IK ver.1.5
- modifications 
	- In tELTPostgresqlMap_10  before it was  sip_traficpeage_n=COALESCE ( sip.sip_trafic_n, 0)   after COALESCE ( sip.sip_trafic_n, null ) . Because trafic_pg_n for pure comtage sections should be null 
	-  In tELTPostgresqlMap_11  add the condition Having sum ( elt.sip_lngsec_n )>0  to SO,AU,DR,TR  and for DI adding elt.sip_lngsec_n >0 . because for lngsec <= 0 So, AU,.... should not appeare, no sense to have DR with 0 length
	Livraison flux  <mark style="background: #BBFABBA6;">STAR_ORDO_CALCUL_2 ver. 0.9 TMC ver.   0.1.54    On 27/04/2026</mark>
	