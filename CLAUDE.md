# FM27 Role Analyzer — projet d'apprentissage

## Mode de travail (important)
- Je construis ce projet **pas à pas pour apprendre**. Ne m'écris pas le code à ma place.
- Donne-moi l'objectif de l'étape, des indices (classes/méthodes à regarder), puis relis mon code.
- Ne montre une solution complète que si je la demande explicitement.
- Réponses concises et directes, en français.

## Contexte
- Jeu : Football Manager 27 (sortie PC/Mac le 10/11/2026, moteur Unity comme FM26).
- Source de données retenue : **mon propre exportateur par OCR** (captures d'écran de la vue
  d'effectif FM27, lues avec Tesseract). Aucune modification du jeu.
  Les exports HTML/CSV (natif ou plugin) restent possibles plus tard comme importeurs supplémentaires.
- Stack : Java 21, Maven, **Spring Boot 4 dès le départ** (CLI via `CommandLineRunner`), Tess4J (Tesseract),
  java.awt / BufferedImage (prétraitement), Jackson 3 (JSON, package `tools.jackson`), JUnit 5 (spring-boot-starter-test),
  Checkstyle (Google adapté à 4 espaces, `config/checkstyle/checkstyle.xml`) et sortpom. Plus tard : Spring Data JPA (H2 puis PostgreSQL), Spring Web.
- Lecture mémoire externe (JNA + Cheat Engine/Il2CppDumper) : idée de second projet, pas maintenant.

## Architecture (à respecter dès le début ; beans Spring injectés par constructeur)
- `model` : Player, Role, Attribute, Slot (postes)
- `io` : interface `PlayerImporter` avec une implémentation par source
  (`OcrImporter` d'abord ; `HtmlImporter`/`CsvImporter` possibles plus tard) + chargement des rôles (JSON).
  Le reste du code ne doit jamais savoir d'où viennent les données.
- `ocr` : capture, prétraitement d'image, découpe en cellules, appel à Tesseract, validation.
- `scoring` : calculs purs, aucune dépendance à Jsoup, au CSV ni aux fichiers
- `App` : orchestration uniquement

## Modèle de score
- Chaque rôle a des attributs clés et préférables (écran tactique du jeu).
  Les rôles FM26/FM27 diffèrent des anciennes versions : relever la liste dans le jeu.
- Score = moyenne pondérée (clés ×3, préférables ×2), ramenée sur 100.
- Attribut en plage "12-15" (brouillard du scouting) → milieu de la plage ; "-" → absent.

## Feuille de route (ordre : phase 1 → 3 → 2 → 4)
**Phase 0 — Exportateur OCR**
O1. Tess4J installé ; OCR brut d'une cellule d'attribut découpée à la main
O2. Prétraitement (agrandir, gris, inversion, binarisation) + whitelist `0123456789-` + PSM ligne unique ; comparer avec O1
O3. Calibration de la grille (X des colonnes, hauteur de ligne) → découpe automatique d'une ligne
O4. Lecture d'un écran complet → `List<Player>` via `OcrImporter`
O5. Validation (1-20 ou plage "12-15") + rapport des cellules douteuses
O6. Capture automatique (`java.awt.Robot`) + défilement et fusion sans doublons

**Phase 1 — Analyseur de rôles (CLI)**
1. Définir `PlayerImporter` et brancher `OcrImporter`
2. Charger les rôles depuis un JSON (Jackson), relevés dans FM27
3. Calculer le score joueur × rôle
4. Interpréter la colonne Position
5. Tests JUnit (OCR sur images de référence, scoring)
6. Meilleur XI pour une tactique (algorithme hongrois)

**Phase 3 — Recrutement Moneyball**
7. Ajouter valeur et salaire au modèle (colonnes OCR supplémentaires)
8. Indice qualité/prix, shortlist filtrée
9. Détecter les postes faibles du XI, proposer les recrues qui les comblent

**Phase 2 — Suivi de progression**
10. Persistance Spring Data JPA / Hibernate (H2 puis PostgreSQL)
11. Entités : Player, Snapshot (export daté), valeurs d'attributs
12. Requêtes d'évolution (par joueur, par saison, plus fortes hausses)

**Phase 4 — Dashboard web**
13. API REST (spring-boot-starter-web) : effectif, rôles, XI, shortlist, progression
14. Front Angular (consomme l'API REST ; CORS ou proxy en dev, DTO côté Spring)

## Avancement
- Étape en cours : O1
