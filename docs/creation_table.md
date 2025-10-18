# Création des tables

```SQL
CREATE TABLE Region (
    id_Region VARCHAR PRIMARY KEY,
    Code_region INTEGER NOT NULL,
    Nom_region VARCHAR NOT NULL
);

CREATE TABLE Commune (
    id_codedep_codecommune VARCHAR PRIMARY KEY,
    id_Region VARCHAR NOT NULL,
    Code_departement INTEGER NOT NULL,
    Nom_commune VARCHAR NOT NULL,
    PMUN INTEGER,
    PCAP INTEGER,
    Code_commune INTEGER NOT NULL,
    FOREIGN KEY (id_Region) REFERENCES Region(id_Region)
);

CREATE TABLE Bien (
    id_bien INTEGER PRIMARY KEY,
    id_codedep_codecommune VARCHAR NOT NULL,
    No_voie INTEGER,
    BTQ VARCHAR,
    Type_voie VARCHAR,
    Voie VARCHAR,
    Total_piece INTEGER,
    Surface_carrez FLOAT,
    Surface_local INTEGER,
    Type_local VARCHAR,
    FOREIGN KEY (id_codedep_codecommune) REFERENCES Commune(id_codedep_codecommune)
);

CREATE TABLE Vente (
    id_vente INTEGER PRIMARY KEY,
    id_bien INTEGER NOT NULL,
    Date DATE NOT NULL,
    Valeur INTEGER,
    FOREIGN KEY (id_bien) REFERENCES Bien(id_bien)
);
```

