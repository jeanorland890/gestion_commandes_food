1. Le Scénario "Auto-Commande à Table" (In-Situ)
   Le client scanne un QR Code posé sur la table. L'URL contient l'ID de la table.

Identification : Le client n'a pas forcément besoin de créer un compte complet. Une session temporaire suffit.

Liaison Table : La commande est liée à un table_id et une zone_id.

Notification : La commande apparaît directement sur l'écran de la cuisine avec la mention "Table 12".

Paiement : Le client peut payer immédiatement via MoMo/Flooz (pour valider la commande) ou demander à payer en fin de repas (si la société l'autorise).

2. Le Scénario "Commande à Distance" (Livraison)
   Le client accède à l'URL globale de la "Food-place".

Identification : Inscription/Connexion souvent requise pour le suivi.

Logistique : Nécessite une adresse_livraison (Coordonnées GPS ou texte) et un frais_livraison.

Attribution Livreurs :

La commande passe en statut pret_livraison.

Un livreur (interne à la société ou indépendant) accepte la course via une table expeditions.
