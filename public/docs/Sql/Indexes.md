-- ################################################
-- 7. INDEXATION
-- ################################################

CREATE INDEX idx_cmd_societe ON commandes(societe_id);
CREATE INDEX idx_cmd_client ON commandes(client_id);
CREATE INDEX idx_items_cmd ON commande_items(commande_id);
CREATE INDEX idx_exp_livreur ON expeditions(livreur_id);
CREATE INDEX idx_tables_statut ON tables(societe_id, statut);