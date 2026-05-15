1. Scan & Identification : L'entrée en session
   Le client arrive à table et scanne le QR Code.

Logique DB : L'URL contient id_table. Le système vérifie l'état dans la table tables.

Vérification :

Si OUI (Déjà occupée) : Le système propose de "Rejoindre la session en cours". Le client voit alors le panier déjà rempli par ses amis.

Si NON (Libre) : Le système initialise une nouvelle commande avec le statut panier_ouvert et marque la table comme occupee.

Isolation : Le societe_id garantit que le client ne voit que le menu du restaurant où il se trouve.

2. Commande Groupée & Panier Partagé (Multi-Utilisateurs)
   Plusieurs personnes collaborent sur la même commande. Une seule personne peut tout gérer, ou chacun peut contribuer.

Modélisation : Une seule commande_id pour la table. Chaque ligne dans commande_items enregistre un client_session_id et un pseudonyme (ex: "Moussa").

Collaboration : Un "Leader" peut être désigné, mais le système permet à n'importe qui d'ajouter des articles.

Temps réel : Grâce au Broadcast, si Moussa ajoute un "Bissap", l'icône du panier s'anime instantanément sur les téléphones de tous les autres convives avec la mention : "Moussa a ajouté 1 Bissap".

3. Workflow Cuisine (KDS - Kitchen Display System)
   Le passage du virtuel au réel.

Flux : Une fois le panier validé (bouton "Envoyer en cuisine"), les items passent au statut en_attente.

Notification : Une alerte sonore retentit en cuisine. Le chef voit le récapitulatif groupé par table. Dès qu'il clique sur "Accepter", le statut passe à en_preparation.

Feedback : Le client voit une barre de progression : "Le chef prépare vos plats...".

4. L'Appel Serveur Digital : Le bouton d'assistance
   Besoin d'aide sans interrompre l'ambiance du restaurant.

Table dédiée : demandes_service.

Champs clés : id, societe_id, table_id, type_demande ('eau', 'serveur', 'addition', 'propreté'), statut.

Action : Le serveur reçoit une notification Push immédiate. L'interface "Staff" affiche une pastille rouge sur la table concernée avec l'icône du besoin.

5. Ajout Successif : La "deuxième tournée"
   La commande n'est pas figée tant que la table n'est pas libérée.

Logique : Le client (ou l'un de ses amis) ajoute de nouveaux produits. Le système les lie à la même commande_id.

Marquage : Ces nouveaux items reçoivent le tag complement.

Priorisation : En cuisine, ces items clignotent ou apparaissent en haut de liste pour signaler que les clients sont déjà à table et attendent ce complément rapidement.

6. Plat Prêt & Service : La synchronisation
   Action : Le cuisinier termine un plat et le marque comme pret.

Notifications simultanées :

Au Serveur : Alerte visuelle pour récupérer le plat au passe-plat.

Au Client : Notification "Votre [Nom du Plat] est prêt et arrive à table !".

Validation : Le serveur livre le plat et clique sur "Servi" sur sa tablette/téléphone pour clôturer le cycle de l'item.

7. Paiement Autonome vs Caisse : La clôture
   Cas A (Digital) : Le client paie via MoMo/Flooz directement depuis l'app. Le Webhook de l'agrégateur valide la transaction, insère une ligne dans paiements et passe la commande en payé.

Cas B (Espèces) : Le client clique sur "Payer en cash". La commande passe en attente_encaissement. Le serveur reçoit une alerte pour se rendre à la table avec l'addition.

Libération : Dès que le montant total est soldé, la table repasse en libre ou a_nettoyer, libérant le QR Code pour les clients suivants.

8. Indisponibilité "Flash" : La sécurité des stocks
   Logique : Le gérant désactive un produit (ex: rupture de stock).

Temps réel : Le produit se grise instantanément sur tous les écrans clients actifs via le Realtime Listener.

Sécurité : Si un client avait déjà mis le produit dans son panier avant la désactivation, au moment de valider, le système effectue une dernière vérification et bloque la commande avec un message : "Désolé, ce produit vient d'être épuisé".
