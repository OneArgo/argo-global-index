# VI - Homogeneous list of fields

## New list

`argo-one-prof-index.csv`

Le fichier commence par une en-tete commentee (lignes prefixees `#`),
alignee sur le format des index Argo actuels (`ar_index_global_prof.txt`
etc.), suivie de la ligne d'en-tete des colonnes puis des donnees :

```
# Title: directory file of the Argo Global Data Assembly Center
# Description : The directory file describes individual files of the Argo GDAC (index homogene et detaille, voir docs/spec-technique.md)
# Project: ARGO
# Index format version: 3.1.0
# Date of update: 2026-09-17 08:23:53
# gdac https root 1: https://data-argo.ifremer.fr
# gdac https root 2: https://nrlgodae1.nrlmry.navy.mil/pub/outgoing/argo
# GDAC node: coriolis
file,type,date,latitude,longitude,date_min,date_max,lat_min,lat_max,lon_min,lon_max,ocean,profiler_type,institution,parameters,parameter_data_mode,parameter_quality,ad_psal_adjustment_mean,ad_psal_adjustment_deviation,date_creation,date_update,gdac_date_update,n_levels,max_pressure,ice_detection,parking depth
```

Champs de l'en-tete :

- `Title` / `Description` / `Project` : texte fixe, identique aux index actuels.
- `Index format version` : version du **format** de cet index (distincte de la version de la specification elle-meme). `3.0.0` designe le format historique (8 colonnes, `ar_index_global_prof.txt`) ; `3.1.0` designe ce nouveau format homogene et detaille (26 colonnes). A incrementer si de nouveaux champs sont ajoutes/retires par la suite.
- `Date of update` : date de generation du fichier (UTC, `YYYY-MM-DD HH:MM:SS`), calculee automatiquement a l'ecriture.
- `gdac https root 1/2` : racines HTTPS permettant de reconstituer une URL complete a partir du chemin relatif de la colonne `file` (ex: `https://data-argo.ifremer.fr/<file>`).
- `GDAC node` : nom du noeud GDAC ayant genere ce fichier (ex: `coriolis`), pas a confondre avec `institution` (DAC proprietaire de chaque fichier individuel, colonne par ligne).
- `type codes` : legende decrivant chaque code de la colonne `type` (source unique : `rules.TYPE_DESCRIPTIONS`, pour rester synchronisee avec le code). Le suffixe `D` (deep) s'ajoute a `PC`/`PB`/`PS` uniquement (ex: `PCD`), pas aux autres types.

**Format des dates** : tous les champs de date (`date`, `date_min`, `date_max`, `date_creation`, `date_update`, `gdac_date_update`, ainsi que `Date of update` dans l'en-tete) sont produits au format **ISO 8601 UTC** (`YYYY-MM-DDTHH:MM:SSZ`, ex: `2023-09-19T13:27:46Z`), et non plus le format compact `YYYYMMDDHHMMSS` utilise par les index historiques (voir les exemples de la section precedente, herites du document de specification d'origine, qui restent au format compact pour fidelite a la source).

Ces lignes sont ignorees par un lecteur CSV standard configure avec
`comment="#"` (voir `readers.existing_index.read_existing_index`), donc
un outil qui ne les interprete pas explicitement peut simplement les
sauter.

**Chemin de la colonne `file`** : toujours relatif a la racine du GDAC
(le repertoire `dac`), jamais un chemin absolu du systeme de fichiers qui
a genere l'index (ex: `aoml/13857/profiles/D13857_001.nc`, jamais
`/scale/project/.../dac/aoml/13857/profiles/D13857_001.nc`). Cela rend le
fichier reutilisable independamment de l'endroit ou il a ete produit, et
compatible avec les racines `gdac https root` ci-dessus pour reconstruire
une URL.

with `type` =

- `'PC'` for R/D core
- `'PB'` for BD/BR files
- `'PS'` for SD/SR files
- `'MPC'` for multiprofile files for 'core'
- `'MPS'` for multiprofile files for 'synthetic'
- `'M'` for metadata file
- `'TE'` for tech file
- `'TR'` for traj file

### A few complementary considerations

**DEEP**: when the float is a deep float (from `profiler_type` indication) then: `type` = concatenation of 'type when not deep' and 'D'. For instance, the deep core profile files type would be 'PCD'.

**POSITION**: with latitude and longitude displaying 4 digits in the decimal part when available (request from a user in POKaPOK).

Up to now, the index fields were containing only information within the file that was indexed and not cross-reference information from other files. With this superindex, a question arose for the new fields `ice_detection`, and at a smaller extent for date and spatial coverage.

**ICE DETECTION**: Additional suggestion at the ADMT meeting was to consider adding a flag for whether the float detected ice. It could be achieved by retrieving the same information as the one used to display the parameter `TECH_FLAG_IceDetection_NUMBER` in the FleetMonitoring (e.g. with 6903258), when it exists. A void field would mean the parameter does not exist, a zero would mean ice was never detected. As the information is within the technical file, it makes sense to add it in the lines associated with 'TE' type. However, considering Romain's comment, the need from the Polar WG was more on having this information for each profile (i.e. associated with 'PC', 'PB', 'PS' types), but so far profile files do not include this information. The polar WG are currently discussing the possibility to add such a parameter within trajectory files and meta file. Thus how and for which file type we record this information depend on:

- Is the polar group need on profile-by-profile?
- Shall we make cross-reference information in the index: i.e. using information from tech file to fill in the `ice_detection` field for index lines regarding profile files?

**DATES**: `date_update` and `date_creation` refer to the associated fields inside the netCDF file, `gdac_date_update` refers to the last file change as stated by the `stat` command/modify date.

**date_min, date_max, lat_min, lat_max, lon_min, lon_max**:

- Same comment as for `ice_detection`: shall we make cross-reference with multiprofile information to fill in these fields for meta related lines?
- Special care when filling in `lon_min` and `lon_max` for floats crossing the 180° lines (Pacific and ACC floats). We need to define the rule we want to use here (it could be extending the longitude domain within [-180, 360] for such floats or use a `lon_min` greater than `lon_max` — that would indicate the 180° cross — or any other solution?)

## Empty fields and specific information with respect to "type"

The following table sums up the fields that would remain empty as not applicable to the file type:

| File Type | Empty fields | Specific information |
| --- | --- | --- |
| PC, PS | `date_min, date_max, lat_min, lat_max, lon_min, lon_max, ice_detection` | `max_pressure` = max( PRES where PRES_QC in (1,2,5,8) ) |
| PB | `date_min, date_max, lat_min, lat_max, lon_min, lon_max, ad_psal_adjustment_mean, ad_psal_adjustment_deviation, ice_detection` | `max_pressure` = max(PRES) |
| MPC, MPS | `date, latitude, longitude, ocean, parameter_data_mode, parameter_quality, ad_psal_adjustment_mean, ad_psal_adjustment_deviation, n_levels, ice_detection` | `max_pressure` = max( PRES where PRES_QC in (1,2,5,8) ) |
| M | `date, latitude, longitude, date_min, date_max, lat_min, lat_max, lon_min, lon_max, ocean, parameter_data_mode, parameter_quality, ad_psal_adjustment_mean, ad_psal_adjustment_deviation, n_levels, max_pressure, ice_detection` | Shall we use `date, latitude, longitude` to mention `launch_date, launch_lat, launch_lon`? |
| TE | `date, latitude, longitude, date_min, date_max, lat_min, lat_max, lon_min, lon_max, ocean, parameter_data_mode, parameter_quality, ad_psal_adjustment_mean, ad_psal_adjustment_deviation, n_levels, max_pressure` | `ice_detection` = 1 if at least once the relevant technical parameter (`TECH_FLAG_IceDetection_NUMBER` for nke floats) mentions that ice was detected. `ice_detection` = 0 if the relevant technical parameter was found and never indicated that ice was detected. `ice_detection` = '' if the relevant technical parameter was not found. |
| TR | `date, latitude, longitude, ocean, parameter_data_mode, parameter_quality, ad_psal_adjustment_mean, ad_psal_adjustment_deviation, n_levels, ice_detection` | `parameter` is `TRAJECTORY_PARAMETERS` field content. `max_pressure` = max( PRES where PRES_QC in (1,2,5,8) ) |

## Let's construct a few examples with each "type"

Reference line (current index format):

```
aoml/7901106/profiles/D7901106_043.nc,20230919132746,22.166,-156.422,P,846,AO,20230922130615,B,B,,-0.005,0.000,20230922203741,20230922203741,496
```

| Type | Example | Additional size |
| --- | --- | --- |
| 'PC' | `aoml/7901106/profiles/D7901106_043.nc,PC,20230919132746,22.1657,-156.4218,,,,,,,P,846,AO,PRES TEMP PSAL,DDD,ABB,-0.005,0.000,20230922203741,20230922130615,20230922203741,496,1599.1,` | + 37 bytes compared to `argo_profile_detailled_index.txt` |
| 'PCD' | `aoml/7901137/profiles/R7901137_010D.nc,PCD,20240323150743,-51.8984,88.7911,,,,,,,I,874,AO,PRES TEMP PSAL,AAA,AAA,,,20240325104028,20240327020138,20240327024046,523,4005.2,` | + 38 bytes compared to `argo_profile_detailled_index.txt` |
| 'PB' | `aoml/1900722/profiles/BD1900722_001.nc,PB,20061022021624,-40.3160,73.3890,,,,,,,I,846,AO,PRES TEMP_DOXY BPHASE_DOXY DOXY,RRRD,B,,,20120520122644,20200312153230,20200725074645,71,2000.0,` | + 59 bytes compared to `argo_bio-profile_index.txt` |
| 'PS' | `aoml/7901108/profiles/SR7901108_012.nc,PS,20240707010139,-40.7230,126.2518,,,,,,,I,846,AO,PRES TEMP PSAL DOXY CHLA BBP700 PH_IN_SITU_TOTAL NITRATE DOWN_IRRADIANCE380 DOWN_IRRADIANCE443 DOWN_IRRADIANCE490 DOWNWELLING_PAR,AAAAAAAARRRR,AAAAAABAAAAA,,,20240709020639,20240709020639,20240709020639,554,1699.3,` | + 55 bytes compared to `argo_synthetic-profile_detailled_index.txt` |
| 'MPS' | `aoml/1901379/1901379_Sprof.nc,MPS,,,,20091106152133,20131105033522,21.853,24.912,-170.12,-157.979,846,AO,PRES TEMP PSAL DOXY NITRATE,,,,,20240629213017,20240629213017,20240629213019,,1298.1,` | + 111 bytes compared to `argo_sprof_index.txt` |
| 'M' | `aoml/7901137/7901137_meta.nc,M,,,,,,,,,,874,AO,TEMP PSAL PRES,,,,,20240710215609,20240710215609,20240710224155,,` | + 62 bytes compared to `ar_index_global_meta.txt` |
| 'TE' | `aoml/1901379/1901379_tech.nc,TE,,,,,,,,,,,AO,,,,,,,,20210428220054,20210428220054,20210428224520,,,` | + 53 bytes compared to `ar_index_global_tech.txt` |
| 'TE' with ice detected | `coriolis/6903258/6903258_tech.nc,TE,,,,,,,,,,,IF,,,,,,,,20230927090759,20240627232909,20240628003621,,,1` | + 54 bytes compared to `ar_index_global_tech.txt` |
| 'TR' | `bodc/3901578/3901578_BRtraj.nc,TR,,,,20230316203450,20240109112930,,,,,836,BO,PRES C1PHASE_DOXY C2PHASE_DOXY TEMP_DOXY DOXY RAW_DOWNWELLING_IRRADIANCE380 RAW_DOWNWELLING_IRRADIANCE412 RAW_DOWNWELLING_IRRADIANCE490 RAW_DOWNWELLING_PAR DOWN_IRRADIANCE380 DOWN_IRRADIANCE412 DOWN_IRRADIANCE490 DOWNWELLING_PAR VRS_PH PH_IN_SITU_FREE PH_IN_SITU_TOTAL FLUORESCENCE_CHLA BETA_BACKSCATTERING700 FLUORESCENCE_CDOM CHLA BBP700 CDOM TEMP_NITRATE TEMP_SPECTROPHOTOMETER_NITRATE HUMIDITY_NITRATE UV_INTENSITY_DARK_NITRATE UV_INTENSITY_DARK_NITRATE_STD FIT_ERROR_NITRATE UV_INTENSITY_NITRATE NITRATE PPOX_DOXY,,,,,20230519163111,20240110013415,20240126221144,,2020.7,` | + 79 bytes compared to `argo_bio-traj_index.txt` |

## Additional size

Here are the various sizes as of 4th April 2024, with estimated increase due to additional fields:

| Index file | Number of lines (without header lines) | Actual size | Additional fields | New size | Chars/line — Min | Chars/line — Max | Chars/line — Ave |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `argo_profile_detailled_index.txt` | 2,954,944 | 417,535,602 Bytes | + 37 × 2,954,944 = + 109,332,928 bytes | 526,868,530 Bytes | 99 + 37 | 218 + 38 | 140.3 + 37 = 177.3 |
| `argo_synthetic-profile_detailled_index.txt` | 308,330 | 48,635,006 Bytes | + 55 × 308,330 = + 16,958,150 bytes | 65,593,156 Bytes | 93 + 55 | 336 + 55 | 156.7 + 55 = 211.7 |
| `argo_bio-profile_index.txt` | 309,541 | 87,952,947 Bytes* | + 59 × 309,541 = + 18,262,919 bytes** | 106,215,866 Bytes | 91 + 59** | 1338 + 59** | 283.1 + 59** = 342.1 |
| **OneArgoIndex-prof.csv (hypothetical new index)** | **3,572,815** | — | — | **698,677,552 Bytes** | **99 + 37** | **1338 + 59** | **194.99** |
| `ar_index_global_traj.txt` | 20,498 | 1,710,671 Bytes | + 79 × 20,498 = + 1,619,342 bytes | 3,330,013 Bytes | 56 + 79 | 89 + 79 | 82.4 + 79 = 99.4 |
| `ar_index_global_Btraj.txt` | 103 | 20,848 Bytes | + 79 × 103 = + 8,137 bytes | 28,985 Bytes | 89 + 79 | 593 + 79 | 196.5 + 79 = 198.5 |
| **OneArgoIndex-traj.csv (hypothetical new index)** | **20,601** | **1,731,519 Bytes** | — | **3,358,998 Bytes** | **56 + 79** | **593 + 79** | **99.9** |
| **TO BE CONTINUED with S files, tech and metadata files** | — | — | — | — | — | — | — |

\* Despite one less field, the size of `argo_bio-profile_index.txt` is much greater than `argo_synthetic-profile_detailled_index.txt` because the intermediate parameters are also listed in there, whereas they are not in the synthetic index.

\*\* It depends on the number of parameters; low value provided (3 parameters).

## Etat d'implementation (a jour, voir historique du depot)

Tous les types sont maintenant generes et valides sur un echantillon reel
(3 flotteurs : TS, BGC, deep) : `PC`/`PCD`, `PB`/`PBD`, `PS`/`PSD`, `TE`,
`M`, `MPC`/`MPS`, `TR`. Le suffixe deep `D` ne s'applique qu'aux types de
profil individuel (`PC`/`PB`/`PS`), pas a `MPC`/`MPS` (fichiers agregeant
plusieurs cycles, potentiellement de profondeurs variees).

Restent en TODO : calcul precis de `ocean` (regle de longitude
approximative actuelle) et `parking depth` (necessite les parametres de
configuration du flotteur).

## Rebuild quotidien a l'echelle du GDAC (~4M fichiers)

Deux mecanismes complementaires, tous deux geres par
`build_index_from_gdac` (voir build_index.py) :

- **Mode incremental** (`--previous-index <index_de_la_veille.csv>`) :
  chaque fichier est compare a la ligne correspondante de l'index
  precedent via `gdac_date_update` (mtime). Si identique, la ligne est
  recopiee sans rouvrir le fichier netCDF (seul un `os.stat`, ~1000x
  moins cher qu'un `open` netCDF, est effectue). Seuls les fichiers
  nouveaux ou modifies depuis la veille sont reellement relus.
- **Parallelisme par flotteur** (`--workers N`) : chaque flotteur est
  independant, traite dans un processus separe
  (`concurrent.futures.ProcessPoolExecutor`). A combiner avec le mode
  incremental : les flotteurs sans fichier modifie retournent quasi
  instantanement, seuls ceux avec du travail reel occupent un worker.

Voir index-argo-hpc.slurm pour un exemple d'utilisation combinee sur un
job SLURM (conserve l'index de la veille, le passe en --previous-index,
--workers aligne sur --cpus-per-task).
