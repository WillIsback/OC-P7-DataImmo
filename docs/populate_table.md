# Peupler les tables

## Table Region

```sql
INSERT INTO region (id_region, code_region, nom_region)
SELECT DISTINCT 
    field1 -- id_region,
    field2 -- code_region,
    field3 -- nom_region
FROM region_temp;
```

## Table Commune

```sql
INSERT INTO commune (id_codedep_codecommune, id_region, code_departement, code_commune, nom_commune, pmun, pcap)
SELECT DISTINCT
    field1 || '-' || field2 as id_codedep_codecommune,
    'R' || printf('%02d', CAST(field3 AS INTEGER)) as id_region,  -- Padding avec 2 chiffres
    field1,
    field2,
    field4,
    field5,
    field6
FROM commune_temp;
```

## Table Bien

```sql
INSERT INTO bien (id_bien, id_codedep_codecommune, no_voie, btq, type_voie, voie, total_piece, surface_carrez, surface_local, type_local)
SELECT

	ROWID, -- id_bien
	field1 ||'-' ||field2 as id_codedep_codecommune, --id_codedep_codecommune
	field3, --no_voie
	field4 || field5 || field6 as btq, --btq
	field7, 
	field8, 
	field9,
	field10,
	field11,
	field12
	
FROM bien_temp
	
```

## Table Vente

```sql
INSERT INTO vente (id_vente, id_bien, date, valeur)
SELECT

	ROWID, -- id_vente,
	ROWID, -- id_bien
	field1,
```

