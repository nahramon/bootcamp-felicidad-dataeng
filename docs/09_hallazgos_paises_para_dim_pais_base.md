# Hallazgos de países para dim_pais_base (se actualiza a medida que avanza el EDA S6b)

## Banco Mundial (266 códigos)
- 207 códigos comparten código con WGI. 59 solo están en Banco Mundial: 49 agregados (regiones, grupos de ingreso, etc.) y 10 territorios.
- Agregados (no son países, tipo = agregado): AFE, AFW, ARB, CEB, CSS, EAP, EAR, EAS, ECA, ECS, EMU, EUU, FCV, HIC, HPC, IBD, IBT, IDA, IDB, IDX, INX ("Not classified"), LAC, LCN, LDC, LIC, LMC, LMY, LTE, MEA, MIC, MNA, NAC, OED, OSS, PRE, PSS, PST, SAS, SSA, SSF, SST, TEA, TEC, TLA, TMN, TSA, TSS, UMC, WLD.
- Territorios solo en Banco Mundial (con ISO3 válido, WGI no los trae): CHI (Islas del Canal), CUW (Curazao), FRO (Islas Feroe), GIB (Gibraltar), IMN (Isla de Man), MAF (San Martín, parte francesa), MNP (Marianas del Norte), SXM (Sint Maarten), TCA (Islas Turcas y Caicos), VGB (Islas Vírgenes Británicas). Pendiente: ver cuáles aparecen en WHR.
- Territorios con datos pero fuera de WHR probable: Islas Vírgenes de EE.UU. (VIR), Macao (MAC), Hong Kong (HKG; WHR sí trae Hong Kong).

## WGI (216 economías, sin agregados)
- 207 códigos coinciden con Banco Mundial. Los 9 que solo están en WGI: AIA (Anguilla), ANT (Antillas Neerlandesas, disuelta en 2010: código retirado, sus sucesores son CUW/SXM/BES), COK (Islas Cook), GUF (Guayana Francesa), JEY (Jersey), MTQ (Martinica), NIU (Niue), REU (Reunión), TWN ("Taiwan, China").
- TWN es el único caso relevante: Banco Mundial no lo incluye, WGI sí, y WHR lo trae como "Taiwan Province of China". Hay que cruzarlo por nombre y asignarle TWN a mano.
- "Réunion" llega con la tilde rota (codificación): cruzar por código, no por nombre.
- No hay códigos viejos (ROM, ZAR, TMP, WBG): ambas fuentes usan los mismos para países. Kosovo (XKX) y Cisjordania/Gaza (PSE) coinciden.

## WHR (2.116 filas, 14 años: 2011, 2012 y 2014-2025; falta 2013)
- WHR no trae código: se cruza por nombre. De los nombres distintos, 22 no coinciden exactamente con ningún nombre de Banco Mundial ni de WGI. El resto (incluido Kosovo) coincide exacto.
- Equivalencias manuales candidatas (código a confirmar contra Banco Mundial/WGI):
  - Congo → COG; DR Congo → COD; Côte d’Ivoire → CIV (apóstrofo curvo ’ en WHR); Egypt → EGY; Gambia → GMB
  - Hong Kong SAR of China → HKG; Iran → IRN; Kyrgyzstan → KGZ; Puerto Rico → PRI; Republic of Korea → KOR
  - Republic of Moldova → MDA; Slovakia → SVK; Somalia → SOM; State of Palestine → PSE; Swaziland → SWZ
  - Syria → SYR; Taiwan Province of China → TWN (solo WGI lo trae); Türkiye → TUR; Venezuela → VEN; Yemen → YEM
- Sin ISO3 (excepción justificada, no son países reconocidos con código ISO): North Cyprus (9 filas, 2012-2021) y Somaliland Region (4 filas, 2011-2015).
- Las columnas de descomposición (whiskers, "Explained by", Dystopia) están vacías en ~52% de las filas; ver por año en el EDA de WHR.

### WHR · confirmación de equivalencias (verificado contra Banco Mundial en Databricks)
- Los 19 códigos candidatos existen en Banco Mundial y los nombres corresponden al país de WHR: CIV, COD, COG, EGY, GMB, HKG, IRN, KGZ, KOR, MDA, PRI, PSE ("West Bank and Gaza"), SOM, SVK, SWZ ("Eswatini"), SYR, TUR, VEN, YEM.
- TWN no está en Banco Mundial (esperado): se confirma contra WGI ("Taiwan, China").
- Total: 20 equivalencias manuales confirmadas + 2 sin ISO3 (North Cyprus, Somaliland Region).

### WHR · descomposición por año
- Whiskers, "Explained by" y Dystopia están vacíos en el 100% de las filas de 2011-2018 y casi completos de 2019 a 2025 (faltan 3 filas de descomposición en 2023 y 1-3 de Dystopia en 2022-2025).
- Consecuencia: el puntaje (Life evaluation) cubre 2011-2025, pero la descomposición oficial de WHR solo sirve de 2019 en adelante.

## OWID (owid_cantril_ladder: 2.270 filas, 178 entidades; owid_gdp_happiness: 7.594 filas, 225 entidades)
- `Code` de OWID es ISO3 para países, sin nulos ni vacíos. Hay 23 entidades cuyo código no está en Banco Mundial:
  - 19 agregados (no son países): OWID_AFR, OWID_ASI, OWID_EU27, OWID_EUR, OWID_HIC, OWID_LIC, OWID_LMC, OWID_NAM, OWID_OCE, OWID_SAM, OWID_UMC, OWID_WRL y WB_EAP, WB_ECA, WB_LAC, WB_MENAP, WB_NA, WB_SA, WB_SSA.
  - Kosovo = OWID_KOS → en Banco Mundial/WGI es XKX (equivalencia manual).
  - Taiwan = TWN (código ISO3 estándar; solo WGI lo trae además de OWID/WHR).
  - Northern Cyprus = OWID_CYN y Somaliland = OWID_SML: códigos propios de OWID, no ISO3. Son los mismos casos que en WHR (North Cyprus, Somaliland Region) → lista de países sin ISO3 justificada.
- Redundancias para Silver: el puntaje de gdp_happiness (2.270 valores no vacíos) coincide en cantidad y rango con cantril_ladder, y "GDP per capita" (510,8 a 174.569,5) coincide con NY.GDP.PCAP.PP.KD de Banco Mundial. OWID sirve de cruce, no de fuente nueva.

### OWID · comparación con WHR y Banco Mundial (verificado en Databricks)
- Puntaje: cantril_ladder = WHR en las 1.948 filas que cruzan por (nombre, año), 0 diferencias. gdp_happiness = cantril_ladder en las 2.270 filas con puntaje (0 sin par, 0 distintas).
- PIB per cápita de OWID vs NY.GDP.PCAP.PP.KD de Banco Mundial: 7.445 filas con PIB; 6.959 cruzan; 1 sola difiere; 486 sin dato en Banco Mundial (probablemente agregados, Taiwán, Kosovo, Chipre del Norte, Somalilandia: pendiente de confirmar).
- Nombres que OWID escribe distinto de WHR (OWID → WHR, con ISO3 de OWID como puente): Cote d'Ivoire → Côte d’Ivoire (CIV); Democratic Republic of Congo → DR Congo (COD); Hong Kong → Hong Kong SAR of China (HKG); Laos → Lao PDR (LAO); Moldova → Republic of Moldova (MDA); Northern Cyprus → North Cyprus (OWID_CYN); Palestine → State of Palestine (PSE); Russia → Russian Federation (RUS); Somaliland → Somaliland Region (OWID_SML); South Korea → Republic of Korea (KOR); Taiwan → Taiwan Province of China (TWN); Turkey → Türkiye (TUR); Vietnam → Viet Nam (VNM).
- WHR "Swaziland" no tiene contraparte por nombre en OWID; su ISO3 (SWZ) se confirma solo contra Banco Mundial ("Eswatini").
- cantril_ladder incluye 11 agregados (Africa, Asia, Europe, North America, Oceania, South America, World, y 4 grupos de ingreso). Ningún país real de OWID falta en WHR (todos aparecen con otro nombre).
- Conclusión: OWID no aporta datos nuevos. Su valor es como puente nombre→ISO3 para WHR (WHR no trae código) y como chequeo cruzado de calidad.

## UNDP (206 filas = 195 países/territorios + 11 agregados)
- Los "~206 países" de la documentación son en realidad 195 países y territorios más 11 agregados.
- Agregados (iso3 con formato propio, hdicode y region vacíos): ZZA.VHHD, ZZB.HHD, ZZC.MHD, ZZD.LHD (grupos de desarrollo humano), ZZE.AS, ZZF.EAP, ZZG.ECA, ZZH.LAC, ZZI.SA, ZZJ.SSA (regiones UNDP) y ZZK.WORLD. Tipo = agregado, excluir de dim_pais_base.
- Los 195 códigos de país existen todos en Banco Mundial: UNDP usa ISO3 estándar y no hay códigos distintos que resolver.
- `region` de UNDP no sirve para dim_pais: está vacía en los 43 países "Very High" sin región (ej. Andorra) más 1 territorio y los 11 agregados. Usar la región de Banco Mundial/WGI.
- 2 filas con hdicode "Other Countries or Territories" (sin HDI 2023). Pendiente: identificar cuáles son.
- Sin Taiwán en UNDP.

## ILOSTAT (ilostat_luu: 18.972 filas; ilostat_neet: 13.829 filas; 276 `ref_area` en cada tabla)
- 98 de los 276 códigos no existen en Banco Mundial:
  - 87 son agregados (X01 a X99 y XA1: regiones, grupos de ingreso, etc.). Tipo = agregado, excluir de dim_pais_base.
  - ANT (Antillas Neerlandesas): código retirado, excluir.
  - CHA = Islas del Canal → en Banco Mundial es CHI (equivalencia manual).
  - KOS = Kosovo → en Banco Mundial/WGI es XKX (equivalencia manual).
  - TWN = Taiwán (solo LUU, 69 filas): ISO3 estándar; Banco Mundial no lo trae.
  - COK (Islas Cook), ESH (Sahara Occidental), FLK (Malvinas), NIU (Niue), REU (Reunión), TKL (Tokelau), WLF (Wallis y Futuna): territorios con ISO3 válido, sin datos en Banco Mundial; probablemente fuera de WHR.
- Los 178 códigos restantes coinciden con Banco Mundial.
- Convención de sexo: SEX_T (total), SEX_M, SEX_F (y SEX_O solo en NEET, 9 filas).
