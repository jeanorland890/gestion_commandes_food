-- ################################################
-- 1. EXTENSIONS & TYPES (ENUMS)
-- ################################################

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Statuts des commandes (In-Situ + Livraison)
CREATE TYPE status_commande AS ENUM (
    'panier_ouvert',       -- En cours de saisie par le client
    'validee',             -- Envoyée en cuisine / En attente validation resto
    'en_preparation',      -- Le chef cuisine
    'pret',                -- Prêt à être servi ou prêt pour le livreur
    'en_livraison',        -- Le livreur a récupéré le colis
    'livre',               -- Remis au client (distant)
    'servi',               -- Remis sur la table (in-situ)
    'paye',                -- Transaction terminée
    'attente_encaissement',-- Cas du paiement cash à table
    'annule'               -- Commande rejetée ou annulée
);

CREATE TYPE status_table AS ENUM ('libre', 'occupee', 'a_nettoyer', 'reservee');
CREATE TYPE type_service AS ENUM ('a_table', 'emporter', 'livraison');
CREATE TYPE type_demande_service AS ENUM ('eau', 'serveur', 'addition', 'propreté');

-- Statuts spécifiques à l'expédition
CREATE TYPE status_expedition AS ENUM (
    'recherche_livreur', 
    'en_route_vers_resto', 
    'recupere', 
    'en_chemin', 
    'arrive_destination', 
    'livre'
);