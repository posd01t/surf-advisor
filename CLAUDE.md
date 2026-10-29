# Ola Pierre — Surf Advisor

PWA perso qui recommande **où, quand et avec quelle planche** surfer depuis Ustaritz (Pays Basque). Interface en français, pensée mobile (installée sur l'écran d'accueil iPhone).

## Stack & déploiement
- **Zéro build, zéro dépendance** : tout tient dans `index.html` (~2 700 lignes : HTML + CSS + JS inline). Police Inter via Google Fonts.
- `manifest.json` : PWA « Ola Pierre », thème sombre `#0b1120`, icône emoji 🏄.
- `sw.js` : service worker, cache `olapierre-vNN` (**incrémenter `CACHE_NAME` à chaque déploiement**, sinon l'iPhone garde l'ancienne version). Static = cache-first, appels open-meteo = network-first avec fallback cache.
- Hébergé sur **GitHub Pages** (déploiement = push sur la branche par défaut).

## Données
- **Open-Meteo** (gratuit, sans clé) : Marine API (houle, période, direction, temp. eau) + Forecast API (vent, rafales, météo), récupérés **par spot** (lat/lon séparés par des virgules). Appels multi-modèles en arrière-plan pour le badge de confiance.
- **Marées** : calculées localement par harmoniques (M2, S2, N2, K2, K1, O1, P1, Q1, M4), coefficient de marée calculé aussi. Pas d'API marée.
- **localStorage** : `olapierre_cache` (cache des prévisions), `olapierre_feedback` (journal des sessions). ⚠️ Le feedback n'existe que sur le téléphone : l'exporter régulièrement (bouton export) avant tout changement de téléphone/navigateur.

## Contenu métier (dans `index.html`)
- `SPOTS` (8) : Uhabia, Ilbarritz, Bidart Centre, Côte des Basques, Marinella, La Madrague, Cenitz, Hendaye. Chaque spot : lat/lon, `driveMin` depuis Ustaritz, `difficulty`, `swellFactor` (exposition), `offshoreDir`, `windProtection`, `bestTide`, `tideExclusion` (règles de sécurité marée + raison), lien webcam Quiksilver. Spots perso ajoutables par l'utilisateur.
- `BOARDS` (quiver, 6) : Hybride 6'4, Longboard 9', Mousse 7'8, Mousse 6', Fourth Reload 6'10, Fish 5'11 Christenson. Chaque planche : volume, `waveRange`, stabilité, maniabilité, `crowdSafe`.
- Curseur **forme du jour** (1–5) qui borne la taille de vague max et exclut les spots difficiles.

## Moteur de score
`computeSpotScore()` combine : taille de vague locale (houle × `swellFactor`), période, vent (offshore/onshore, rafales > 40 km/h = exclusion, cap offshore), chop de mer du vent, alignement marée, direction de houle, distance, tendance de houle, affluence par spot/créneau, confiance (écart-type absolu entre modèles). Exclusions de sécurité via `isSpotExcluded()`.
`findBestSlot()` cherche le meilleur créneau dans la fenêtre de dispo ; `recommendBoard()` choisit la planche ; `recommendWetsuit()` la combi selon la temp. de l'eau.

## UI
Onglets jour (J à J+4) avec badge de confiance, barre Top 3, cartes spot (score, conditions, mini-courbe de marée SVG, webcam), spots exclus avec raison, timeline horaire, filtre de disponibilité (slider double + presets combinables, lever/coucher du soleil), fraîcheur de la prévision, modal de feedback (date, durée, planche, tags, mode « pas à l'eau ») + historique + export JSON pour recalibrer le scoring.

## Historique (résumé du git log)
- Avr. 2026 : v1, feedback, filtre de dispo, courbe de marée, spots perso, fix marée inversée.
- Juin 2026 : calibration du scoring (protection vent, cambrure, distance dynamique), bannière de confiance, mode « pas à l'eau », webcams.
- Sept. 2026 : prévisions par spot, confiance multi-modèles sur 5 jours, chop, tendance de houle, affluence par spot, Fish 5'11 ajoutée.

## Conventions de travail
- Garder le fichier unique sans build ; tester sur mobile (iPhone) en priorité.
- Messages de commit courts en anglais ; textes de l'UI en français.
- Toute modif du scoring : la justifier par les feedbacks exportés quand c'est possible.
