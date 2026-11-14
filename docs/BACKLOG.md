# Backlog

Statuts : `TODO` · `EN COURS` · `DONE`

## v1 — Extraction des attributs de mon effectif

**Objectif de la version :** à partir d'un dossier de captures des fiches joueurs de mon effectif,
l'outil enregistre un relevé daté (snapshot) de chaque joueur et de ses attributs dans PostgreSQL.

**Critère de fin :** ≥ 99 % de cellules correctes sur ~5 fiches de référence, cellules douteuses corrigées avant enregistrement.

### US-00 — Mise en place du projet · `EN COURS`
En tant que développeur, je veux un projet Spring Boot outillé et versionné, pour travailler proprement dès le départ.
- [x] Dépôt Git, `.gitignore` Java/Maven/IDE
- [x] `pom.xml` : Spring Boot 4, Java 21, Tess4J, Jackson 3, Checkstyle (4 espaces), sortpom
- [ ] `App` (`@SpringBootApplication`) et packages `model`, `io`, `ocr`, `scoring` (avec `package-info.java`)
- [ ] `application.properties` : chemin `tessdata`, surchargeable par variable d'environnement
- [ ] Test `contextLoads` ; `mvn verify` passe
- [ ] Maven Wrapper (`mvnw`) et `.gitattributes` (fins de ligne LF)
- [ ] `README.md` : but de l'outil, prérequis (Java 21, Tesseract, Docker), commandes
- [ ] GitHub Actions : `mvn verify` sur chaque PR ; PR bloquée si échec
- [ ] Tesseract installé et fonctionnel sur Ubuntu et Windows

### US-01 — OCR brut (O1) · `TODO`
En tant que développeur, je veux lire une cellule d'attribut avec Tess4J, pour valider la chaîne technique.
- [ ] Une cellule découpée à la main est lue par Tess4J depuis Java
- [ ] Résultat brut noté (base de comparaison pour US-02)

### US-02 — Prétraitement (O2) · `TODO`
En tant que développeur, je veux améliorer la lecture d'une cellule, pour fiabiliser l'OCR.
- [ ] Agrandissement, niveaux de gris, inversion, binarisation (`BufferedImage`)
- [ ] Whitelist `0123456789-` et PSM ligne unique pour les attributs ; `fra` sans whitelist pour le texte
- [ ] Comparatif avant / après sur plusieurs cellules
- [ ] Tests unitaires du prétraitement

### US-03 — Conditions de capture et fiches de référence · `TODO`
En tant qu'utilisateur, je veux des conditions de capture figées et des données de référence, pour mesurer la qualité de l'OCR.
- [ ] `docs/FICHE_OCR.md` rempli : résolution, zoom, skin, langue, mises en page champ et gardien
- [ ] Liste exacte des attributs FM27 relevée → `enum Attribute` (libellé FR, catégorie)
- [ ] ~5 fiches PNG dans `src/test/resources/fiches/` (dont un gardien, un nom accentué, des attributs à 1 chiffre)
- [ ] Un `xxx.expected.json` écrit à la main par fiche

### US-04 — Gabarits de la fiche (O3) · `TODO`
En tant que développeur, je veux décrire les zones de la fiche, pour découper automatiquement chaque valeur.
- [ ] `gabarit-champ.json` et `gabarit-gardien.json` (coordonnées en pixels) dans `src/main/resources`
- [ ] Chargement des gabarits avec Jackson
- [ ] Mode `calibrate` : dessine les zones sur une capture pour vérification visuelle

### US-05 — Une fiche → un joueur (O4) · `TODO`
En tant qu'utilisateur, je veux qu'une capture devienne un `Player`, pour obtenir ses données.
- [ ] Détection automatique du gabarit (champ / gardien)
- [ ] Lecture : nom, date de naissance, postes (brut), pied préféré, attributs
- [ ] Test : taux de cellules correctes calculé sur les fiches de référence (objectif ≥ 99 %)

### US-06 — Docker et PostgreSQL (D1) · `TODO`
En tant que développeur, je veux une base PostgreSQL locale, pour stocker les relevés.
- [ ] Cours `/teach` : Docker, Docker Compose, PostgreSQL vs MySQL
- [ ] `compose.yaml` avec PostgreSQL ; démarrage par Spring Boot (`spring-boot-docker-compose`)
- [ ] Connexion vérifiée avec `psql`

### US-07 — Schéma Flyway (D2) · `TODO`
En tant que développeur, je veux un schéma versionné, pour faire évoluer la base proprement.
- [ ] Cours `/teach` : Flyway
- [ ] `V1__init.sql` : `save`, `player`, `snapshot`, valeurs d'attributs
- [ ] Contrainte d'unicité (sauvegarde, nom, date de naissance) et (sauvegarde, date de jeu) pour le snapshot

### US-08 — Persistance JPA (D3) · `TODO`
En tant que développeur, je veux enregistrer un snapshot, pour garder l'historique.
- [ ] Cours `/teach` : Spring Data JPA, entités, repositories, transactions
- [ ] Entités et repositories ; enregistrement d'un snapshot en une transaction
- [ ] Joueur existant (clé naturelle) réutilisé, sinon créé

### US-09 — Tests d'intégration (D4) · `TODO`
En tant que développeur, je veux tester la persistance sur un vrai PostgreSQL, pour éviter les surprises.
- [ ] Cours `/teach` : Testcontainers
- [ ] Tests repository / service avec Testcontainers ; passent aussi en CI

### US-10 — Validation et correction (O5) · `TODO`
En tant qu'utilisateur, je veux corriger les valeurs douteuses, pour ne rien enregistrer de faux.
- [ ] Règles : attribut entier 1-20, nom non vide, date de naissance valide
- [ ] Pour chaque doute : affichage joueur, attribut, valeur lue, chemin de l'image découpée ; saisie de la correction
- [ ] Rien n'est enregistré tant qu'il reste des doutes

### US-11 — Commande d'import (O6) · `TODO`
En tant qu'utilisateur, je veux importer un dossier de captures, pour créer un relevé de mon effectif.
- [ ] `PlayerImporter` / `OcrImporter` ; le reste du code ignore la source
- [ ] `import --save "Lens" --game-date 2027-08-01 --dir captures/ [--replace]`
- [ ] Snapshot en double refusé sans `--replace`
- [ ] Export JSON du snapshot (debug)
- [ ] Résumé en fin d'import (joueurs lus, corrections, durée)

## Après la v1 (ordre ajustable selon le besoin)

### Capture automatique · `TODO`
*`java.awt.Robot` sur le PC de jeu, bouton « joueur suivant ». Stories à écrire.*

### v2 — Score par rôle et meilleur XI
*Relever les rôles FM27 dans le jeu (avec / sans ballon) → `roles.json`. Stories à écrire.*

### v3 — Suivi de progression
*Comparaison de snapshots. Stories à écrire.*

### v4 — API REST et front Angular
*Spring Web + `frontend/` Angular. Stories à écrire.*

### v5 — Recrutement Moneyball
*Fiches hors effectif : plages "12-15", valeur, salaire. Stories à écrire.*
