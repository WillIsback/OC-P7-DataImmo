# Data Analyse

## Nb total de vente premier semestre 2020

```sql
SELECT 
	COUNT(id_vente) as nb_vente
FROM vente, bien
WHERE
	vente.id_bien = bien.id_bien AND
	lower(type_local)="appartement" AND
	date BETWEEN '2020/01/01' AND '2020/06/30';

```

## Le nombre de ventes d’appartement par région pour le 1er semestre 2020

```sql
SELECT 
    COUNT(id_vente) as nb_vente,
    nom_region
FROM vente, bien, commune, region
WHERE
    vente.id_bien = bien.id_bien AND
    bien.id_codedep_codecommune = commune.id_codedep_codecommune AND
    commune.id_region = region.id_region AND
    LOWER(type_local) = 'appartement' AND
    date BETWEEN '2020/01/01' AND '2020/06/30'
GROUP BY nom_region
ORDER BY nb_vente DESC;
```

## Proportion des ventes d’appartements par le nombre de pièces

```sql
WITH total_ventes AS (
    SELECT COUNT(*) as total
    FROM vente, bien
    WHERE vente.id_bien = bien.id_bien 
    AND LOWER(type_local) = 'appartement'
)
SELECT 
    COUNT(id_vente) as nb_vente,
    total_piece,
    ROUND(COUNT(id_vente) * 100.0 / total_ventes.total, 2) as proportion_pct
FROM vente, bien, total_ventes
WHERE
    vente.id_bien = bien.id_bien AND
    LOWER(type_local) = 'appartement'
GROUP BY total_piece
ORDER BY nb_vente DESC;
```

## Liste des 10 départements où le prix du mètre carré est le plus élevé.

```sql
SELECT 
    code_departement,
    ROUND(valeur/surface_carrez) as prix_m2
FROM bien, vente, commune
WHERE 
		vente.id_bien = bien.id_bien AND
		bien.id_codedep_codecommune = commune.id_codedep_codecommune
GROUP BY code_departement
ORDER BY prix_m2 DESC
LIMIT 10;
```

## Prix moyen du mètre carré d’une maison en Île-de-France

```sql
SELECT 
    avg(valeur/surface_carrez) as prix_m2
FROM bien, vente, commune, region
WHERE 
	vente.id_bien = bien.id_bien AND
	bien.id_codedep_codecommune = commune.id_codedep_codecommune AND
	commune.id_region = region.id_region AND
	lower(type_local)="maison" AND
	lower(nom_region) LIKE "ile-de-france";

```

## Liste des 10 appartements les plus chers avec la région et le nombre de mètres carrés

```sql
SELECT 
    bien.id_bien,
	vente.valeur,
    bien.surface_carrez,
	region.nom_region
FROM bien, vente, commune, region
WHERE 
	vente.id_bien = bien.id_bien AND
	bien.id_codedep_codecommune = commune.id_codedep_codecommune AND
	commune.id_region = region.id_region AND
	lower(type_local)="appartement"
	
ORDER BY valeur DESC
LIMIT 10;

```

## Taux d’évolution du nombre de ventes entre le premier et le second trimestre de 2020.

```sql
WITH total_S1_ventes AS (
    SELECT COUNT(*) as total
    FROM vente
    WHERE date BETWEEN '2020/01/01' AND '2020/03/31'
),
total_S2_ventes AS (
    SELECT COUNT(*) as total
    FROM vente
    WHERE date BETWEEN '2020/04/01' AND '2020/06/30'
)
SELECT
	s1.total as trimestre_1,
	s2.total as trimestre_2,
    ROUND(((s2.total - s1.total) * 100.0 / s1.total), 2) as evolution_pct
FROM total_S1_ventes as s1, total_S2_ventes as s2;
```

## Le classement des régions par rapport au prix au mètre carré des
appartement de plus de 4 pièces.

```sql
SELECT 
    region.nom_region,
    ROUND(AVG(valeur/surface_carrez), 2) as prix_m2_moyen
FROM vente, bien, commune, region
WHERE
    vente.id_bien = bien.id_bien AND
    bien.id_codedep_codecommune = commune.id_codedep_codecommune AND
    commune.id_region = region.id_region AND
    LOWER(bien.type_local) = 'appartement' AND
    bien.total_piece > 4
GROUP BY region.nom_region
ORDER BY prix_m2_moyen DESC;
```

## Liste des communes ayant eu au moins 50 ventes au 1er trimestre

```sql
SELECT
    commune.nom_commune,
    COUNT(vente.id_vente) as nb_vente
FROM vente, bien, commune
WHERE
    vente.id_bien = bien.id_bien AND
    bien.id_codedep_codecommune = commune.id_codedep_codecommune AND
    vente.date BETWEEN '2020/01/01' AND '2020/03/31'
GROUP BY commune.nom_commune
HAVING nb_vente > 50
ORDER BY nb_vente DESC;
```

## Différence en pourcentage du prix au mètre carré entre un
appartement de 2 pièces et un appartement de 3 pièces.

```sql
WITH prix_m2_2p AS (
    SELECT
        AVG(vente.valeur/bien.surface_carrez) as avg
    FROM vente, bien
    WHERE 
        vente.id_bien = bien.id_bien AND
        bien.total_piece = 2 AND
        LOWER(bien.type_local) = 'appartement'
),
prix_m2_3p AS (
    SELECT
        AVG(vente.valeur/bien.surface_carrez) as avg
    FROM vente, bien
    WHERE 
        vente.id_bien = bien.id_bien AND
        bien.total_piece = 3 AND
        LOWER(bien.type_local) = 'appartement'
)
SELECT
    ROUND(p2.avg, 2) as prix_m2_2p,
    ROUND(p3.avg, 2) as prix_m2_3p,
    ROUND(((p3.avg - p2.avg) / p2.avg) * 100, 2) as difference_pct
FROM prix_m2_2p p2, prix_m2_3p p3;
```

## Les moyennes de valeurs foncières pour le top 3 des communes des départements 6, 13, 33, 59 et 69.

Option 1 → Top 3 global

```sql
SELECT 
	ROUND(AVG(vente.valeur), 2) as vente_avg,
	commune.nom_commune,
	commune.code_departement
FROM
	vente,
	bien,
	commune
WHERE
	vente.id_bien = bien.id_bien AND
	bien.id_codedep_codecommune = commune.id_codedep_codecommune AND
	commune.code_departement IN (6, 13, 33, 59, 69)
GROUP BY commune.nom_commune
ORDER BY vente_avg DESC
LIMIT 3
```

Option 2 → Top 3 par département

```sql
WITH classement AS (
    SELECT 
        ROUND(AVG(vente.valeur), 2) as valeur_moy,
        commune.nom_commune,
        commune.code_departement,
        ROW_NUMBER() OVER (PARTITION BY commune.code_departement ORDER BY AVG(vente.valeur) DESC) as rang
    FROM vente, bien, commune
    WHERE
        vente.id_bien = bien.id_bien AND
        bien.id_codedep_codecommune = commune.id_codedep_codecommune AND
        commune.code_departement IN (6, 13, 33, 59, 69)
    GROUP BY commune.nom_commune, commune.code_departement
)
SELECT valeur_moy, nom_commune, code_departement
FROM classement
WHERE rang <= 3
ORDER BY code_departement, rang;
```

## Les 20 communes avec le plus de transactions pour 1000 habitants pour les communes qui dépassent les 10 000 habitants.

```sql
SELECT 
	commune.ptot,
	commune.nom_commune,
	ROUND((COUNT(vente.id_bien) * 1000 / commune.ptot), 2) as vente_per_1000_hab
FROM
	vente, bien, commune
WHERE
	vente.id_bien = bien
```
