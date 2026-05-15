--
-- Table: societes
--
CREATE TABLE societes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    nom TEXT NOT NULL,
    ifu TEXT UNIQUE,
    telephone TEXT,
    email TEXT,
    adresse TEXT,
    ville TEXT,
    logo_url TEXT,
    latitude DECIMAL(9,6),  -- Position du resto pour calcul frais livraison
    longitude DECIMAL(9,6),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
--

-- Table: clients
--
CREATE TABLE clients (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    nom TEXT NOT NULL,
    prenom TEXT,
    telephone TEXT UNIQUE NOT NULL, -- Identifiant clé au Bénin
    email TEXT,
    mot_de_passe_hash TEXT,         -- Pour les comptes persistants (Livraison)
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
--

-- Table: adresses_clients
--
CREATE TABLE adresses_clients (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    client_id UUID NOT NULL REFERENCES clients(id) ON DELETE CASCADE,
    label TEXT NOT NULL,           -- ex: "Maison", "Bureau"
    details TEXT,                  -- "Derrière l'église, portail bleu"
    latitude DECIMAL(9,6), 
    longitude DECIMAL(9,6),
    est_principale BOOLEAN DEFAULT FALSE
);
--

-- Table: utilisateurs
--
CREATE TABLE utilisateurs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    nom TEXT NOT NULL,
    role TEXT CHECK (role IN ('admin', 'gerant', 'serveur', 'cuisine')),
    email TEXT UNIQUE NOT NULL,
    est_actif BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
--

-- Table: categories
--
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    nom TEXT NOT NULL,
    ordre_affichage INTEGER DEFAULT 0
);
--

-- Table: produits
--
CREATE TABLE produits (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    categorie_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    nom TEXT NOT NULL,
    prix_unitaire NUMERIC(10, 2) NOT NULL,
    est_disponible BOOLEAN DEFAULT TRUE
);
--

-- Table: zones
--
CREATE TABLE zones (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    nom TEXT NOT NULL
);
--

-- Table: livreurs
--
CREATE TABLE livreurs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    nom TEXT NOT NULL,
    telephone TEXT UNIQUE NOT NULL,
    zone_id UUID REFERENCES zones(id) ON DELETE SET NULL,
    est_disponible BOOLEAN DEFAULT TRUE
);
--

-- Table: tables
--
CREATE TABLE tables (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    zone_id UUID REFERENCES zones(id) ON DELETE CASCADE,
    numero_table TEXT NOT NULL,
    statut status_table DEFAULT 'libre',
    UNIQUE (societe_id, zone_id, numero_table)
);
--

-- Table: commandes
--
CREATE TABLE commandes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    table_id UUID REFERENCES tables(id), 
    client_id UUID REFERENCES clients(id),    -- NULL si commande anonyme à table
    adresse_id UUID REFERENCES adresses_clients(id),
    frais_livraison NUMERIC(10, 2) DEFAULT 0,
    serveur_id UUID REFERENCES utilisateurs(id),
    code_court TEXT,                          -- ex: #102
    statut status_commande DEFAULT 'panier_ouvert',
    type_service type_service DEFAULT 'a_table',
    origine_commande TEXT CHECK (origine_commande IN ('qr_code', 'web_distant')),
    montant_total NUMERIC(10, 2) DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
--

-- Table: commande_items
--
CREATE TABLE commande_items (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    commande_id UUID NOT NULL REFERENCES commandes(id) ON DELETE CASCADE,
    produit_id UUID NOT NULL REFERENCES produits(id),
    client_session_id TEXT,       -- Indispensable pour Scénario 1 (Moussa a ajouté...)
    nom_client_affiché TEXT,      -- Pseudonyme temps réel
    quantite INTEGER NOT NULL,
    prix_unitaire_applique NUMERIC(10, 2) NOT NULL,
    notes TEXT,
    est_complement BOOLEAN DEFAULT FALSE, -- Tag "Ajout successif"
    statut_item TEXT DEFAULT 'en_attente'
);
--

-- Table: expeditions
--
CREATE TABLE expeditions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    commande_id UUID UNIQUE NOT NULL REFERENCES commandes(id) ON DELETE CASCADE,
    livreur_id UUID REFERENCES livreurs(id),
    statut_exp status_expedition DEFAULT 'recherche_livreur',
    code_otp_validation TEXT,     -- Code à donner au livreur pour confirmer la remise
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
--

-- Table: demandes_service
--
CREATE TABLE demandes_service (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    table_id UUID NOT NULL REFERENCES tables(id) ON DELETE CASCADE,
    type_demande type_demande_service NOT NULL,
    statut TEXT DEFAULT 'en_attente'
);
--

-- Table: paiements
--
CREATE TABLE paiements (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    societe_id UUID NOT NULL REFERENCES societes(id) ON DELETE CASCADE,
    commande_id UUID NOT NULL REFERENCES commandes(id) ON DELETE CASCADE,
    methode_paiement TEXT CHECK (methode_paiement IN ('momo', 'flooz', 'cash', 'carte')),
    montant_paye NUMERIC(10, 2) NOT NULL,
    transaction_ref TEXT,        -- ID MoMo / Flooz
    statut_paiement TEXT DEFAULT 'valide'
);
--
-- Fin du schéma
--