# 08 · Hallazgos del EDA de bronze (S6b)

EDA de 6 pasos (estructura, nulos, cardinalidad, rangos, duplicados, temporal) sobre las tablas bronze, un notebook por fuente (`N_bronze_eda_<fuente>`, D26). Las consultas están comentadas en cada notebook; acá queda el resumen y las reglas que Silver (S7) y `dim_pais_base` (S6) tienen que aplicar. Los hallazgos de países, para armar `dim_pais_base`, están en `claude/09_hallazgos_paises_para_dim_pais_base.md`.

Estado: hecho para Banco Mundial, WGI, WHR, OWID, UNDP e ILOSTAT. **Open-Meteo queda pendiente** (la descarga sigue incompleta por el rate limit de la API, ver D22).

## Resumen por fuente

| Fuente | Hallazgo principal | Regla para Silver |
|---|---|---|
| Banco Mundial (398.468 filas, 266 códigos, 1.498 indicadores) | 0 duplicados, 0 claves vacías. Celdas vacías = `NULL` real (no el texto `"null"`). Valores extremos revisados y explicables (Palau CO2, hiperinflaciones, Kuwait 1991...): se conservan. 49 agregados regionales mezclados con países y 10 territorios. `EG.FEC.RNEW.ZS` termina en 2022 y `SH.STA.SUIC.P5` en 2021: con arrastre de 2 años no cubren el final del panel. | `UNPIVOT` + filtro contra `config_indicadores`; `CAST` directo (0 valores no numéricos); sacar los 49 agregados. |
| WGI (22.540 filas, 216 economías, 4 dimensiones) | Formato largo. 0 duplicados en (economía, dimensión, año). Estimado y puntaje sin faltantes; los `..` solo están en las ~40 columnas por fuente, que no se usan. Años 1996, 1998, 2000 y 2002-2025 (faltan 1997, 1999, 2001). 9 territorios sin región. | Quedarse con estimado/puntaje de las 4 dimensiones incluidas; descartar columnas por fuente. |
| WHR (2.116 filas, 14 años: 2011, 2012, 2014-2025) | Falta 2013. `Year` y `Rank` vienen como texto con `.0`. Sin código de país (se cruza por nombre). Puntaje 1,364-7,856. Whiskers y descomposición ("Explained by", Dystopia) vacíos al 100% en 2011-2018 y casi completos desde 2019. | `CAST` a DOUBLE y luego INT para año y rank. La descomposición oficial solo sirve de 2019 en adelante. |
| OWID (cantril 2.270 filas; gdp_happiness 7.594 filas) | Puntaje idéntico a WHR (1.948 de 1.948 filas cruzadas, diferencia máxima 0,0005). PIB idéntico a Banco Mundial (6.958 de 6.959). `gdp_happiness` repite el puntaje de `cantril_ladder` (2.270 de 2.270). No trae países reales que WHR no tenga; sí 11 agregados. | No pasa a Silver como datos (D24). `cantril_ladder` sirve de puente nombre→ISO3 y de chequeo cruzado en S8. |
| UNDP (206 filas = 195 países + 11 agregados; 1.115 columnas) | 36 indicadores con serie por años (30 de 1990-2023, 6 de 14 años). 0 duplicados, 0 no numéricos. Los 195 iso3 existen en Banco Mundial. `region` de UNDP vacía en 43 países Very High, 1 territorio y los 11 agregados. `config_indicadores` no tenía filas de UNDP. | Registrar los 6 indicadores (D25). Despivotar solo las columnas de los 6 indicadores (con `spark.sql` + f-strings, semana 6). Sacar los 11 agregados (`ZZA.VHHD`...`ZZK.WORLD`). Región: usar la de Banco Mundial. |
| ILOSTAT (luu 18.972 filas; neet 13.829 filas) | 0 duplicados; cada país-año-sexo viene de una sola fuente (no hace falta elegir fuente). Sexo: SEX_T/M/F (+ SEX_O en 9 filas de NEET). `obs_status` es `NULL` real o una marca: `R` en 4.118 filas de luu (aparece en todos los años 2005-2024, entre 13% y 35% de los países por año, y en ninguna fila de 2025-2027; significado exacto por confirmar con la lista de códigos de ILOSTAT), `B` (quiebre de serie) en 632 y `U` en 29 de neet. 14 `obs_value` nulos en neet. LUU: 276 `ref_area`, 2005-2027, casi completo. Los años 2026 y 2027 son futuros y 2025 trae exactamente los mismos 271 países que 2026 y 2027 (todos sin marca): son casi seguro estimaciones modeladas, no observaciones. NEET: 1970-2027, esparso; en 2026-2027 solo trae los 87 agregados. | Filtrar `sex = 'SEX_T'`; sacar los 87 agregados `X..` y `ANT`; conservar `obs_status` como columna; no usar 2026-2027 (años futuros) y decidir si 2025 entra como estimación o se corta en 2024; decidir qué hacer con las filas `R`; unir luu + neet recién en Silver (D16). |
| Open-Meteo | Pendiente (descarga incompleta, D22). | — |

## Coberturas para el filtro D3 (a revisar en Silver)
- ILOSTAT NEET: restando los 87 agregados, quedan aproximadamente 94 países con dato en 2011, 122 en 2019 y 87 en 2025 (supone que los agregados están en todos los años; no verificado fila por fila). Es consistente con D4 (excepción de cobertura) pero hay que declararlo.
- UNDP `gii`: ~79% en 2011 (cerca del umbral de 80%), ~88% en 2023; si no pasa D3, `gdi` es el reemplazo (D25). UNDP termina en 2023: con arrastre de 2 años cubre justo hasta 2025.
- Redundancia a evaluar (Spearman ≥ 0,95): UNDP `le` vs `SP.DYN.LE00.IN`; UNDP `gnipc` vs `NY.GDP.PCAP.PP.KD`; `hdi` es combinación de `le`, `eys`, `mys` y `gnipc`.

## Pendientes
- Confirmar el significado de `obs_status = 'R'` en `ilostat_luu` con la lista de códigos de ILOSTAT (por año ya se descartó que sean las proyecciones recientes: está en 2005-2024 y no en 2025-2027).
- Decidir el año de corte de ILOSTAT LUU en Silver (2024 observado o 2025 como estimación).
- Identificar qué países tiene OWID/Banco Mundial para los 486 valores de PIB sin par (probablemente agregados y territorios).
- EDA de Open-Meteo cuando termine la descarga.
- Armar `dim_pais_base` (S6) con los hallazgos de `claude/09_hallazgos_paises_para_dim_pais_base.md`.
