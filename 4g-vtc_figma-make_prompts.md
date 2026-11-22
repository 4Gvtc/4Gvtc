# 4G VTC — Prompts Figma Make (fusion SwiftRide / 4G VTC / PRD)

## Mode d'emploi (lis ça avant de coller quoi que ce soit)

1. Colle **PROMPT 0** seul. Attends la fin de la génération.
2. Vérifie la checklist « Validation » à la fin du prompt. Si un point échoue, corrige avec un message court avant de continuer.
3. Colle ensuite **un seul prompt de phase à la fois**, dans l'ordre. Ne les enchaîne jamais sans validation : chaque phase suppose que la précédente tient.
4. Avant de commencer, importe ton **vrai logo 4G VTC** (Toyota Corolla, bleu) dans Figma Make. Les icônes du zip SwiftRide sont celles de l'ancienne marque, ne les réutilise pas.

### Décisions que j'ai prises à ta place (conteste-les maintenant, pas après la génération)

- **Marque = dégradé vertical bleu `#2563eb` (haut) vers bleu marine `#0b1f4d` (bas).** Le bleu seul sur fond sombre fait 3,9:1 de contraste, insuffisant pour du texte courant (WCAG exige 4,5:1). Un dégradé ne peut servir ni de couleur de texte ni de couleur d'icône : il est réservé aux fonds (boutons, pastille du logo, écran d'accueil). Le texte blanc dessus reste >= 5,2:1 sur toute la hauteur. Le texte bleu sur fond sombre utilise `#6c9bff`. Le bas du dégradé étant presque aussi sombre que le fond de l'app (1,5:1), chaque surface en dégradé a un liseré clair de 1 px pour rester lisible comme composant (WCAG 1.4.11).
- **Les éléments graphiques gardent un bleu uni `#2563eb`** : tracé d'itinéraire, marqueurs, anneaux de focus, graphiques. Un dégradé sur un tracé de carte ou un graphique nuit à la lecture.
- **Pas d'emoji.** Ton prototype utilise des emojis comme icônes (🚗, 💎, 🏍️). Ça rend comme un prototype, pas comme un produit de store. Le prompt impose des icônes SVG.
- **Aucune donnée utilisateur fictive.** Tu as déjà rejeté des profils bidon. Les catalogues (véhicules, chauffeurs, codes promo, lieux de Bordeaux) sont des données de référence légitimes. Historique, statistiques, points de fidélité et notifications démarrent à zéro et sont calculés à partir des courses réellement effectuées dans l'app.
- **Paiement : espèces par défaut** (cohérent avec 4G VTC en Firebase Spark), cartes / Apple Pay / Google Pay / PayPal derrière un `PaymentProvider` en mode stub inactif.
- **Arabe = RTL.** Ça impose des propriétés CSS logiques partout, des icônes directionnelles inversées et une carte non miroir. C'est un coût réel : ne te plains pas si la phase de polish est longue.
- **Seuils inventés (à valider)** : paliers de fidélité, frais d'annulation, attente payante, tarifs au km. Ils sont isolés dans `data/tariffs.ts` et `data/loyalty.ts`, modifiables en un endroit.
- **Services externes** : Nominatim et OSRM sont des serveurs publics qui peuvent échouer ou limiter depuis l'environnement de Figma Make. Le prompt impose un repli local (géocodeur et routage simulés sur Bordeaux).

---

## PROMPT 0 — Fondations (à coller en premier)

```
Tu es à la fois designer produit senior, ingénieur React/TypeScript senior et architecte mobile. Tu construis "4G VTC", une application de VTC (ride-hailing) premium pour Bordeaux, capable de rivaliser avec Uber et Bolt. Ce n'est pas une maquette : chaque écran doit fonctionner de bout en bout sur des données simulées mais cohérentes, derrière des couches d'abstraction remplaçables par un vrai backend plus tard.

Cette première passe pose UNIQUEMENT les fondations : design system, internationalisation, architecture, couche de données simulée, types et règles métier. Ne construis pas encore les écrans métier ; livre seulement une page de démonstration du design system et un écran d'accueil minimal avec la navigation. D'autres prompts suivront, phase par phase.

## 1. Produit
- Nom : "4G VTC". Palier premium futur : "4G+".
- Trois rôles dans une seule app, routes séparées : passager (/), chauffeur (/driver), administrateur (/admin). Prévois un sélecteur de rôle de démonstration discret dans un panneau "Mode démo" (aussi : accélérer la simulation x1/x5/x10, réinitialiser les données).
- Ville de référence : Bordeaux. Centre par défaut [44.8378, -0.5792]. Devise : EUR.
- Mobile-first (390 px de large comme référence). Sur bureau (>1024 px) : deux colonnes, panneau de contrôle de 420 px à gauche, carte plein hauteur à droite.
- Le logo est fourni en import. Crée un composant <Logo /> qui l'utilise, avec un wordmark texte "4G VTC" en repli.

## 2. Règles non négociables
- AUCUN emoji, nulle part (interface, données, alt, code). Icônes : lucide-react, taille et épaisseur de trait cohérentes (20 px / 1.75 par défaut). Les véhicules ont une silhouette SVG dédiée par catégorie.
- AUCUNE donnée utilisateur inventée. Les catalogues (véhicules, chauffeurs, codes promo, lieux de Bordeaux) sont des données de référence. Historique, statistiques, points de fidélité, notifications, cartes enregistrées démarrent vides pour un nouveau compte et se remplissent par l'usage réel. Tout chiffre affiché doit être calculé à partir du store, jamais écrit en dur.
- Aucun texte d'interface en dur dans les composants : tout passe par le système i18n.
- Pas de placeholder creux ni de bouton sans action. Un bouton qui ne fait rien n'existe pas : soit il est branché, soit il est absent.
- Pas de localStorage dans les composants directement : tout passe par la couche de persistance de la section 6.

## 3. Design system
Direction : interface de mobilité haut de gamme, dense en information utile, calme, lisible en plein soleil comme de nuit. Pas de néon, pas de dégradé violet, pas de glassmorphisme sur tout. Le seul dégradé autorisé est le dégradé de marque défini plus bas. Le verre dépoli est autorisé uniquement pour la barre de navigation et les feuilles (sheets) flottant sur la carte.

Jetons de couleur (les deux thèmes ont des valeurs DIFFÉRENTES ; définis-les en variables CSS dans theme.css, avec `:root` pour le clair et `.dark` pour le sombre, et un vrai bascule persisté qui respecte aussi prefers-color-scheme) :

Sombre : --background #07070e ; --surface #0f0f1c ; --surface-2 #171726 ; --foreground #ecedf6 ; --muted-foreground #8c8cb4 ; --border rgba(255,255,255,.08) ; --primary #2563eb ; --primary-foreground #ffffff ; --primary-text #6c9bff (pour tout TEXTE ou icône bleue sur fond sombre) ; --accent #f5e642 ; --accent-foreground #0a0a10 ; --success #34d399 ; --danger #f04060.
Clair : --background #f4f5fb ; --surface #ffffff ; --surface-2 #eceef8 ; --foreground #0a0a18 ; --muted-foreground #55557f ; --border rgba(0,0,0,.09) ; --primary #2563eb ; --primary-foreground #ffffff ; --primary-text #1d4ed8 ; --accent #f5e642 (jamais en texte sur fond clair, uniquement en fond avec texte sombre) ; --success #0f9d6b ; --danger #d12347.

Dégradé de marque (identique dans les deux thèmes) : --brand-gradient: linear-gradient(180deg, #2563eb 0%, #0b1f4d 100%) ; --brand-gradient-edge: 1px solid rgba(255,255,255,.16).
- Il remplace le bleu uni UNIQUEMENT sur des FONDS : boutons primaires, pastille du logo, bouton flottant principal, pastille de l'onglet actif de la barre de navigation, en-tête de la carte "course en cours", écran d'accueil / onboarding (dégradé plein écran, bleu en haut, marine en bas).
- Il ne s'applique JAMAIS à : du texte, des icônes, des bordures, des anneaux de focus, des tracés ou marqueurs de carte, des graphiques. Ces éléments utilisent le bleu uni --primary (#2563eb) ou --primary-text.
- Toute surface en dégradé porte le liseré --brand-gradient-edge (le bas marine se confond presque avec le fond sombre).
- États : au survol, éclaircis le dégradé de 6 % ; à l'appui, applique l'échelle .97 sans changer le dégradé ; désactivé : remplace par --surface-2 avec texte --muted-foreground (ne jamais baisser l'opacité d'un dégradé).
- Le dégradé de marque est le SEUL dégradé de l'application. Aucun autre dégradé décoratif.
- Texte sur dégradé : toujours blanc (--primary-foreground), >= 5.2:1 sur toute la hauteur.

Contrastes : texte courant >= 4.5:1, grand texte et composants graphiques >= 3:1. Le bleu #2563eb ne doit JAMAIS servir de couleur de texte sur fond sombre (3.9:1) : utilise --primary-text. Les boutons primaires ont un texte blanc sur --brand-gradient.

Typographie : Outfit (300-800) pour l'interface latine, JetBrains Mono pour les prix, plaques, codes PIN, minuteurs et identifiants. Pour l'arabe : Noto Sans Arabic. Échelle : 12 / 14 / 16 / 20 / 28 / 40. Chiffres tabulaires (font-variant-numeric: tabular-nums) pour tous les montants. Longueur de ligne < 70 caractères. Pas de libellés en MAJUSCULES systématiques, pas de mot isolé coloré dans un titre.

Formes : rayon 14 px (cartes), 10 px (champs, boutons secondaires), 999 px uniquement pour les puces. Ombres quasi absentes ; la hiérarchie passe par les surfaces et les bordures.

Mouvement (bibliothèque motion/react) :
- Retour immédiat au toucher : l'état pressé (échelle .97) s'applique sur pointerdown, pas au relâchement.
- Ressorts (spring) plutôt que durées fixes, interruptibles : un geste ou un tap peut inverser une animation en cours depuis sa valeur actuelle.
- Feuilles (bottom sheets) glissables avec suivi 1:1 du doigt, inertie et points d'accroche (peek / mi-hauteur / plein).
- Transitions de page courtes (< 250 ms). Aucun mouvement décoratif en boucle sauf l'indicateur de recherche de chauffeur et le point pulsé de la position.
- Respecte prefers-reduced-motion : remplace les translations par des fondus.

## 4. Internationalisation
Langues : fr (défaut), en, de, es, ar. Détection par navigator.language, choix manuel dans le menu, persistance.
- Un fichier JSON par langue dans src/app/i18n/, clés hiérarchiques (booking.confirm.title), pluriels via Intl.PluralRules, interpolation typée. Un test vérifie que toutes les langues ont exactement les mêmes clés.
- Arabe : <html dir="rtl" lang="ar">. Utilise EXCLUSIVEMENT des propriétés logiques (ms-/me-/ps-/pe-/start/end, inset-inline-*, text-start) ; les icônes directionnelles (flèches, chevrons, retour) s'inversent ; la carte, les numéros de téléphone, les plaques et les montants restent en LTR via <bdi> / dir="ltr".
- Formats : Intl.NumberFormat / DateTimeFormat selon la langue (fr-FR, en-GB, de-DE, es-ES, ar avec chiffres latins "ar-u-nu-latn"). Distances en km, heures en 24 h sauf en.
- Prévois +35 % de longueur pour l'allemand : aucune largeur fixe sur des libellés.

## 5. Architecture (dossiers sous src/app)
types/        types de domaine
lib/          id, format, geo (haversine, bbox), fare, time, validators
data/         catalogues de référence : vehicles, drivers, promos, places (Bordeaux), tariffs, loyalty
i18n/         fr.json en.json de.json es.json ar.json + provider + hooks
integrations/ un dossier par service, chacun avec : interface (contrat), mockProvider, index (sélection du provider)
  auth/ store/ geocoding/ routing/ maps/ payments/ dispatch/ chat/ notifications/
features/     booking, tracking, payments, safety, chat, account, loyalty, referral, subscriptions, scheduling, voice, driver, admin, analytics
hooks/        useRide, useGeolocation, useCountdown, useMediaQuery, useReducedMotion, useTheme
components/   ui (shadcn existant) + composants partagés (Sheet, Stat, EmptyState, ErrorState, Skeleton, PriceTag, Avatar, Rating, VehicleGlyph)
Règle : une feature n'importe jamais directement un mockProvider ; elle passe par l'interface exposée dans integrations/<service>/index.

Contrats à définir (interfaces TypeScript) :
- AuthProvider : signUp, signIn, signInAnonymously, signInWithProvider('google'|'apple'), signOut, onAuthChange, deleteAccount.
- DataStore (de type Firestore) : get, list, create, update, delete, subscribe(path|query, callback) avec transactions simples (runTransaction) pour l'acceptation d'une course par un chauffeur.
- GeocodingProvider : search(query, bias), reverse(lat, lon). Implémentation 1 : Nominatim (antirebond 450 ms, limite à 1 requête/s, Bordeaux en biais). Implémentation 2 (repli automatique en cas d'échec réseau) : recherche locale dans data/places.ts.
- RoutingProvider : route(points[]) renvoie { distanceKm, durationMin, geometry, alternatives[] }. Implémentation 1 : OSRM public. Repli : ligne droite pondérée par un facteur 1.35 avec durée à 28 km/h en ville.
- MapProvider : rendu Leaflet + tuiles CARTO (dark_all / light_all selon le thème), marqueurs, polylignes, zoom, recentrage, ajustement aux limites.
- PaymentProvider : listMethods, addMethod, removeMethod, setDefault, charge(amount, rideId). Mode stub inactif : succès simulé, jamais d'appel réel. Moyens : espèces (défaut), carte, Apple Pay, Google Pay, PayPal.
- DispatchService : assignDriver(ride) choisit le chauffeur disponible le plus proche (Haversine), lui envoie la demande avec un délai de 20 s ; sans réponse, passe au suivant.
- ChatService : sendMessage, subscribe(rideId).
- NotificationService : push local (toasts + centre de notifications).

## 6. Couche de données simulée
- Store en mémoire avec persistance (IndexedDB ou localStorage encapsulé dans integrations/store) et diffusion entre onglets via BroadcastChannel : un passager, un chauffeur et un admin ouverts dans trois onglets se voient en temps réel.
- Moteur de simulation : les chauffeurs se déplacent le long de la géométrie de la route à vitesse réaliste, avec bruit GPS léger ; le temps de simulation s'accélère avec le "Mode démo".
- Jeu de départ déterministe : catalogues uniquement, plus des chauffeurs répartis autour du centre de Bordeaux dont l'état (en ligne / hors ligne / en course) est géré par le moteur.

## 7. Types de domaine (src/app/types)
Définis au minimum : User (role: 'rider'|'driver'|'admin'), RiderProfile, DriverProfile (licence, plaque, assurance, statut de vérification, femmes uniquement, niveau, années d'expérience, langues), VehicleType, Ride (statuts : scheduled, searching, driver_assigned, driver_arrived, in_progress, completed, cancelled ; stops[], fare breakdown, surge, promo, tip, paymentMethod, pin, womenOnly, carpool, splitCount, sos), Place, SavedPlace, PaymentMethod, PromoCode, Notification, ChatMessage, Dispute, LoyaltyTier, Subscription, Referral.

## 8. Catalogue de départ (données de référence)
Véhicules (id, nom, groupe, capacité, ETA de base en min, ratio tarifaire par rapport à Standard) :
moto Moto 1 place 1 min ×0.40 ; eco Éco 4 places 5 min ×0.66 ; std Standard 4 places 3 min ×1.00 ; confort Confort 4 places 4 min ×1.38 ; elec Électrique 4 places 6 min ×1.11 (zéro émission) ; xl XL 6 places 7 min ×1.73 ; van Van 8 places 10 min ×2.65 ; prem Premium 4 places 6 min ×2.45 ; luxe Luxe 3 places 9 min ×4.29 ; taxi Taxi 4 places 2 min ×0.84.
Chauffeurs de référence : Antoine M. (Tesla Model 3, noir, BX-241-CD, FR/EN, niveau Elite Pro, 4.97), Fatima B. (Peugeot 508, gris, BX-182-GH, FR/AR, Or, 4.94), Pierre L. (Mercedes EQS, blanc, BX-305-KL, FR/ES, Or, 4.91), Sophie N. (Renault Zoé, rouge bordeaux, BX-078-OP, FR/EN, Elite Pro, 4.99, certifiée "femmes uniquement"). Photos : utilise le composant ImageWithFallback. Les compteurs de courses des chauffeurs sont calculés par le moteur, pas fixes.
Lieux de Bordeaux : Place de la Bourse, Gare Saint-Jean, Cité du Vin, Aéroport de Bordeaux-Mérignac, Cours de l'Intendance, Grosse Cloche, Mériadeck, Chartrons, Place Gambetta, Darwin (avec coordonnées réelles).
Codes promo : RIDE20 (-2,00 EUR sur la prochaine course, une utilisation par compte).

## 9. Règles métier (isole-les dans lib/fare.ts et data/tariffs.ts, avec tests unitaires)
- Tarif Standard = 2,00 EUR + 0,95 EUR/km + 0,22 EUR/min. Chaque catégorie applique son ratio. Tarif minimum Standard 6,00 EUR (× ratio).
- Majoration dynamique (surge) : jours ouvrés 7h-9h et 17h-19h ×1.3 ; vendredi et samedi 22h-3h ×1.5. Affichée de façon explicite avant confirmation.
- "Price lock" : le prix affiché est verrouillé 1 heure.
- Multi-arrêts : jusqu'à 3 arrêts, +1,00 EUR par arrêt.
- Covoiturage : -40 % sur le tarif de base (le matching d'un second passager est simulé et expliqué à l'utilisateur).
- Mode "femmes uniquement" : filtre les chauffeurs certifiés.
- Split fare : diviseur x2/x3/x4, lien de partage (Web Share API avec repli presse-papiers).
- Pourboire : 0 / 1 / 2 / 3 / 5 EUR ou montant libre.
- Attente payante : 3 min gratuites, puis 0,30 EUR/min.
- Annulation : gratuite pendant 2 min après l'assignation, ensuite 3,00 EUR.
- Fidélité par paliers selon le nombre de courses terminées : Bronze (0), Argent (10), Or (30), Platine (75), Elite (150), avec cashback 0 / 1 / 2 / 3 / 5 %.
- Empreinte carbone : estimée par course (g CO2 selon catégorie et distance), cumul visible dans le profil.

## 10. Accessibilité (WCAG 2.1 AA)
HTML sémantique, repères ARIA, navigation clavier complète, focus visible (anneau 2 px --primary-text), cibles tactiles >= 44 x 44 px, pièges de focus corrects dans les feuilles et dialogues, annonces aria-live pour les changements d'état de course, libellés sur tous les champs, messages d'erreur explicites (ce qui s'est passé + pourquoi + comment corriger), alt sur toute image porteuse de sens.

## 11. Performance et qualité
Code splitting par route et par feature (React.lazy), mémoïsation des listes lourdes, squelettes de chargement avec shimmer (jamais de spinner plein écran), composants EmptyState / ErrorState / SuccessState réutilisables, Error Boundaries par feature, TypeScript strict (aucun any), documentation courte dans chaque dossier (README.md).

## 12. Livrable de cette passe
1. Thème clair et sombre RÉELLEMENT distincts, bascule fonctionnelle.
2. Système i18n complet avec les 5 langues pour les clés de base (navigation, boutons communs, erreurs) et bascule RTL fonctionnelle.
3. Toute l'arborescence de la section 5, interfaces et mockProviders compris.
4. Une page /design-system montrant jetons, typographie, boutons, champs, puces, cartes, feuille glissable, états vide / erreur / succès / chargement, dans les deux thèmes et en RTL.
5. Un écran d'accueil passager minimal avec la carte de Bordeaux, la barre de navigation (Accueil, Trajets, Sécurité, Profil) et le panneau "Mode démo".

## Validation (à vérifier avant la phase suivante)
- Le thème clair n'a pas les mêmes valeurs que le sombre.
- Passer en arabe inverse bien la mise en page, sans casser la carte ni les montants.
- Aucun emoji dans le code ni à l'écran.
- Le dégradé bleu vers marine n'apparaît que sur des fonds (boutons, logo, onglet actif, accueil), avec liseré clair ; le tracé de carte et les graphiques restent en bleu uni.
- Aucune chaîne de texte en dur dans les composants.
- Les mockProviders sont importés uniquement via integrations/<service>/index.
```

---

## PROMPT 1 — Parcours de réservation passager

```
Contexte : les fondations (Prompt 0) sont en place. Construis maintenant le parcours de réservation passager complet, en respectant strictement les règles et le design system existants.

Étapes (machine à états explicite dans features/booking, avec feuille glissable sur mobile et panneau latéral sur bureau) :
1. Départ et destination. Champs avec autocomplétion en temps réel (GeocodingProvider, état de chargement, état "aucun résultat", gestion d'erreur réseau avec repli local). Bouton "ma position" (Geolocation API, avec refus géré proprement). Épingles déplaçables sur la carte. Sous les champs : lieux enregistrés (Domicile, Bureau, Favoris), destinations récentes (issues des vraies courses), suggestions de lieux de Bordeaux. Les sections vides affichent un état vide utile.
2. Arrêts multiples : ajout jusqu'à 3 arrêts intermédiaires (réorganisables), +1,00 EUR par arrêt, recalcul de l'itinéraire.
3. Aperçu d'itinéraire : tracé, distance, durée, alternatives sélectionnables, indicateur de trafic estimé selon l'heure.
4. Choix du véhicule : les 10 catégories avec silhouette SVG, capacité, ETA, prix calculé par lib/fare.ts (jamais écrit en dur), badge de majoration si actif, filtre par groupe, filtre "Électrique", options "Covoiturage" (-40 %) et "Femmes uniquement" (masque les chauffeurs non certifiés, désactivé avec explication s'il n'y en a aucune disponible).
5. Confirmation : récapitulatif détaillé du tarif (base, distance, durée, majoration, arrêts, promo, pourboire éventuel, total), code promo (RIDE20 validé par PromoService, erreur claire si invalide ou déjà utilisé), choix du moyen de paiement, split fare x2/x3/x4, programmation optionnelle (voir ci-dessous), bouton "Réserver" affichant le total.
6. Recherche du chauffeur : animation d'attente, dispatch via DispatchService, possibilité d'annuler (frais selon la règle des 2 minutes).
7. Chauffeur trouvé : profil complet (photo, nom, note, nombre de courses, véhicule, plaque, langues, niveau, expérience), code PIN de vérification à 4 chiffres à montrer au chauffeur, compte à rebours d'arrivée basé sur la position simulée, boutons Appeler et Message (le chat est branché en phase 4).
8. Course en cours : suivi en direct (marqueur chauffeur qui suit la géométrie, ETA recalculé), minuteur, tarif en cours, bouton partage de trajet, bouton SOS visible en permanence.
9. Fin de course : récapitulatif, note 1-5 avec étiquettes (Conduite sûre, Ponctuel, Voiture propre, Trajet agréable, Bonne conversation), pourboire, reçu.

Programmation : choix de date et heure (minimum +15 min, maximum 30 jours), course créée au statut "scheduled", rappel par notification 15 min avant, déclenchement du dispatch à l'heure.

Écrans d'état obligatoires pour chaque étape : chargement (squelette), vide, erreur (avec action de reprise), succès.
Clavier : tout le parcours se fait au clavier. Chaque changement de statut est annoncé via aria-live.

Les courses terminées alimentent immédiatement : l'historique, les destinations récentes, le compteur de fidélité et l'empreinte carbone.

Validation : une course complète réservée au départ de la Gare Saint-Jean, avec 1 arrêt, catégorie Confort et code RIDE20, doit afficher un total cohérent avec lib/fare.ts, passer par tous les statuts et apparaître ensuite dans les destinations récentes.
```

---

## PROMPT 2 — Carte, géolocalisation et temps réel

```
Contexte : le parcours de réservation fonctionne. Remplace toute simulation visuelle approximative par une expérience de carte réaliste, derrière MapProvider / RoutingProvider.

- Carte interactive Leaflet, tuiles CARTO dark_all en thème sombre et light_all en thème clair, changement de tuiles à la bascule du thème.
- Marqueurs : départ (point bleu pulsé), destination (point jaune), arrêts intermédiaires numérotés, chauffeur (pastille avec silhouette du véhicule orientée selon le cap).
- Contrôles : zoom +/- (>= 44 px), bouton recentrer, bouton "ma position".
- Position réelle : Geolocation API avec watchPosition, précision affichée par un cercle, refus de permission géré par un message explicatif et un repli sur le centre de Bordeaux.
- Itinéraire : tracé avec halo, alternatives en gris cliquables, ajustement automatique aux limites avec marge pour ne pas passer sous la feuille.
- Chauffeurs à proximité : affichés pendant le choix du véhicule (uniquement ceux réellement disponibles dans le store), avec indicateur de densité par zones.
- Suivi en direct : le chauffeur avance le long de la géométrie ; si l'itinéraire change, le tracé se met à jour.
- Trafic : indicateur estimé par tranche horaire (pas de fausse donnée de trafic en temps réel : indique clairement que c'est une estimation).
- Résilience : si les tuiles, Nominatim ou OSRM échouent, la carte affiche un bandeau discret "Mode hors ligne limité" et bascule sur les replis locaux, sans erreur bloquante.
- La carte ne se miroite jamais en RTL.

Validation : couper le réseau en cours de réservation ne casse ni le parcours ni la carte.
```

---

## PROMPT 3 — Paiements, reçus et abonnements

```
Contexte : branche la couche de paiement et le portefeuille, toujours derrière PaymentProvider (mode stub inactif, aucun appel réel).

- Moyens de paiement : espèces (par défaut), carte bancaire, Apple Pay, Google Pay, PayPal. Ajouter une carte (formulaire avec validation Luhn, date d'expiration, CVC ; on ne stocke que marque + 4 derniers chiffres), la supprimer, définir le moyen par défaut. Les moyens non encore activés côté provider affichent "Bientôt disponible" au lieu de simuler un faux succès.
- Écran Paiement dans le profil + sélecteur dans la confirmation de course.
- Reçus : générés pour chaque course terminée, avec détail du tarif, TVA, moyen de paiement ; export PDF ou impression via une feuille de style dédiée.
- Factures : génération d'une facture numérotée séquentiellement avec les informations du compte.
- Historique des paiements filtrable par mois.
- Split fare : lien de partage généré, les parts payées par les autres passagers sont suivies dans le récapitulatif.
- Codes promo : gestion dans le portefeuille (appliqués, expirés), règles de validation dans PromoService.
- Abonnements "4G+" : plans mensuel et annuel (prix et avantages définis dans data/, par exemple réduction fixe, priorité de dispatch, annulation gratuite étendue), souscription, résiliation avec confirmation explicite des conséquences ("Résilier 4G+ ? Vous perdez la priorité de dispatch à la fin de la période en cours."), état actif visible.
- Parrainage : code personnel, lien de partage, suivi des filleuls et des récompenses réellement acquises (aucun compteur fictif).

Validation : payer une course en espèces, une autre avec une carte ajoutée, produire un reçu et une facture, souscrire puis résilier 4G+.
```

---

## PROMPT 4 — Sécurité, chat et notifications

```
Contexte : ajoute la couche sécurité et communication.

- Centre de sécurité : accès rapide à SOS, partage de trajet, contacts de confiance, services d'urgence (numéros 17, 15, 112 sur appel direct), vérification du chauffeur (plaque, photo, véhicule, statut de vérification), code PIN.
- SOS : bouton toujours visible pendant une course, confirmation en deux étapes pour éviter les fausses alertes, capture de la géolocalisation, enregistrement sur la course (sosActive, coordonnées, horodatage), notification aux contacts de confiance, lien de position partageable (Web Share API).
- Contacts de confiance : ajout (nom, téléphone validé), suppression, option "notifier automatiquement au départ de chaque course".
- Partage de trajet en direct : lien partageable affichant la progression.
- Chat dans l'app (ChatService) : messagerie passager-chauffeur par course, états envoyé / lu, saisie accessible, réponses rapides prédéfinies traduites ("Je suis devant l'entrée", "J'arrive dans 2 minutes"), chat désactivé après la fin de la course avec message explicite.
- Centre de notifications : course assignée, chauffeur arrivé, course démarrée, course terminée, promotions, confirmations de paiement, alertes de sécurité ; non-lus, tout marquer comme lu, suppression ; toasts éphémères en complément. Le compteur de la barre de navigation est calculé, jamais fixe.
- Signalement et litige sur une course terminée (catégorie, description, statut suivi par l'admin).

Validation : déclencher un SOS en course génère l'alerte visible côté admin dans un second onglet.
```

---

## PROMPT 5 — Compte passager, historique et analytique

```
Contexte : complète le compte passager.

- Profil : avatar (import d'image avec recadrage), nom, e-mail, téléphone, langue, thème, préférences (véhicule électrique par défaut, mode femmes uniquement par défaut).
- Onboarding à la création du compte : le nom et le téléphone sont demandés ; aucun profil prérempli.
- Authentification via AuthProvider : e-mail / mot de passe, Google, Apple (états d'erreur propres si indisponible), connexion anonyme avec invitation à créer un compte pour conserver les données.
- Lieux enregistrés : Domicile, Bureau, favoris libres ; ajout, édition, suppression, épingle sur carte.
- Chauffeurs favoris.
- Historique des courses : date, départ, arrivée, chauffeur, catégorie, tarif, note donnée, reçu, bouton "Réserver à nouveau" qui préremplit le parcours. Filtres par mois et par statut, état vide soigné.
- Fidélité : palier courant, progression vers le suivant, cashback, récompenses (calculées à partir du nombre de courses terminées).
- Empreinte carbone : cumul et courses électriques.
- Tableau de bord passager (features/analytics) : nombre total de courses, dépenses du mois, destinations favorites, économies réalisées (promos + cashback), graphiques (recharts) avec description textuelle alternative pour les lecteurs d'écran.
- Vie privée : bannière de consentement aux cookies, pages CGU / confidentialité / mentions légales, export RGPD des données du compte (JSON téléchargeable), suppression de compte avec confirmation explicite des conséquences.

Validation : un compte neuf n'affiche aucun chiffre inventé ; après 3 courses, tous les compteurs reflètent exactement ces 3 courses.
```

---

## PROMPT 6 — Application chauffeur

```
Contexte : construis l'application chauffeur sous /driver, avec la même identité visuelle, entièrement internationalisée (RTL compris).

- Inscription et vérification : formulaire en étapes (identité, âge >= 21 ans calculé à partir de la date de naissance, permis de conduire avec date de validité, véhicule avec plaque, assurance, import de documents simulés). Statut de vérification : en attente, validé, suspendu, refusé ; un chauffeur non validé ne peut pas passer en ligne et voit exactement ce qui manque.
- Écran principal : bascule En ligne / Hors ligne, carte avec position, zone de forte demande, résumé du jour.
- Demandes de course : carte de demande avec compte à rebours de 20 s, départ, destination, distance, gain estimé, accepter / refuser ; l'acceptation passe par une transaction du DataStore pour qu'un seul chauffeur obtienne la course.
- Course : navigation vers le passager, bouton "Arrivé", vérification du PIN du passager, démarrage, fin de course, encaissement (espèces confirmées), attente payante décomptée.
- Chat avec le passager et notation du passager.
- Gains : tableau de bord (gains du jour / de la semaine / du mois, nombre de courses, heures en ligne, note moyenne) calculé uniquement à partir des courses effectuées ; graphiques avec alternative textuelle ; historique des courses ; commission de la plateforme affichée clairement.
- Paramètres : véhicule, langues parlées, certification "femmes uniquement" (soumise à validation admin), documents.

Validation : dans deux onglets (passager et chauffeur), une course réservée côté passager apparaît chez le chauffeur, est acceptée, vérifiée par PIN, terminée, et met à jour les gains et l'historique des deux côtés.
```

---

## PROMPT 7 — Administration

```
Contexte : construis le panneau d'administration sous /admin (accès limité au rôle admin), responsive, internationalisé.

- Vue d'ensemble : courses en cours, chauffeurs en ligne, courses du jour, revenu du jour, note moyenne, alertes SOS actives (en tête, rouge, avec position sur carte), le tout calculé depuis le store.
- Chauffeurs : liste filtrable (en attente, validé, suspendu), fiche détaillée avec documents, boutons Valider / Refuser / Suspendre / Réactiver avec confirmation décrivant les conséquences, et notification envoyée au chauffeur.
- Courses : liste, filtres, détail complet, carte du trajet.
- Litiges et signalements : file d'attente, statut, réponse à l'utilisateur, résolution.
- Codes promo : création, désactivation, suivi d'utilisation.
- Analytique : courses par jour, revenus, taux d'annulation, temps moyen d'assignation, zones de demande, répartition par catégorie, avec tableaux alternatifs accessibles.
- Journal d'audit des actions administratives.

Validation : suspendre un chauffeur en ligne le fait passer hors ligne immédiatement dans l'onglet chauffeur.
```

---

## PROMPT 8 — Fonctions avancées (voix, IA, scheduling avancé)

```
Contexte : ajoute les fonctions avancées, chacune avec un comportement de repli honnête.

- Réservation vocale : bouton micro utilisant la Web Speech API, dans la langue de l'interface. Extraction d'intention de type "emmène-moi à la gare Saint-Jean" via un analyseur local (regex + lieux connus), confirmation orale écrite avant réservation. Si l'API n'est pas disponible (par exemple Firefox), le bouton est absent et la saisie texte reste disponible, sans message d'erreur parasite.
- Suggestions de destinations (nommées "Suggestions pour vous", pas "IA" tant que ce n'est pas de l'IA) : calculées localement à partir de l'heure, du jour et de l'historique réel (par exemple trajet Domicile vers Bureau habituel le matin). Elles n'apparaissent que si l'historique le justifie.
- Filtre véhicules électriques persistant dans les préférences.
- Programmation avancée : liste des courses programmées, modification, annulation avec règle de frais, rappel.
- Prévision de prix et d'ETA : fourchette selon l'heure, basée sur lib/fare.ts et les règles de majoration.

Validation : sans micro autorisé, aucune erreur visible ; sans historique, aucune suggestion.
```

---

## PROMPT 9 — Polish, PWA et audit final

```
Contexte : passe de finition et d'audit sur l'ensemble de l'application.

1. Thème : vérifie chaque écran dans les deux thèmes ; corrige tout contraste inférieur à 4.5:1 (texte) ou 3:1 (composants).
2. RTL : vérifie chaque écran en arabe ; corrige toute propriété physique restante (left/right) par son équivalent logique ; vérifie que les icônes directionnelles s'inversent.
3. Langues : vérifie les débordements en allemand (+35 %) et la complétude des clés dans les 5 langues.
4. États : chaque liste a un état de chargement (squelette avec shimmer), un état vide utile, un état d'erreur avec reprise, un état de succès.
5. Gestes : tirer pour rafraîchir sur les listes, retour haptique via navigator.vibrate (si disponible) sur confirmations et SOS, feuilles glissables interruptibles.
6. Mouvement : audit prefers-reduced-motion.
7. PWA : manifest (nom 4G VTC, thème bleu #2563eb, icônes générées à partir du logo importé, y compris une version maskable), service worker avec mise en cache des ressources statiques et page hors ligne explicite.
8. Performance : code splitting vérifié, images en chargement différé, listes longues virtualisées, aucun re-rendu inutile des composants de carte.
9. Accessibilité : audit WCAG 2.1 AA complet (clavier seul, lecteur d'écran, zoom 200 %, cibles >= 44 px) avec liste des corrections effectuées.
10. Nettoyage : supprime le code mort, les composants shadcn inutilisés, les données en dur restantes et tout texte non traduit.
11. Livre un fichier PROJECT_CONTEXT.md décrivant les décisions d'architecture, les providers et la marche à suivre pour remplacer chaque mockProvider par un vrai service (Firebase, Supabase, Stripe, Google Maps) sans toucher aux features.

Livre à la fin un rapport d'audit : ce qui a été corrigé, ce qui reste, et pourquoi.
```

---

## Ce que ces prompts ne peuvent pas te donner

Figma Make produit un front. Il ne déploie ni Cloud Functions, ni règles Firestore, ni webhooks Stripe, et ne publie pas sur les stores. Le dispatch, les paiements réels et la notification push restent à brancher plus tard derrière les interfaces `integrations/*`. Le fichier `PROJECT_CONTEXT.md` demandé en phase 9 est fait pour ça.
