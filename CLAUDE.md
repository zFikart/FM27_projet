# FM27 Role Analyzer — projet d'apprentissage

## Mode de travail (important)
- Je construis ce projet **pas à pas pour apprendre**. Ne m'écris pas le code à ma place.
- Donne-moi l'objectif de l'étape, des indices (classes/méthodes à regarder), puis relis mon code.
- Ne montre une solution complète que si je la demande explicitement.
- Réponses concises et directes, en français.
- Notions nouvelles (Docker, PostgreSQL, Flyway, JPA, Testcontainers…) : passer par le skill `/teach`.
  Je connais MySQL : expliquer PostgreSQL par différence avec MySQL.
- **Coaching Git / PR** tout au long du projet : à chaque commit, branche, rebase ou PR, me rappeler
  la bonne pratique, relire mes messages de commit, titres et descriptions de PR, et corriger mes
  erreurs (commits trop gros, messages flous, fichiers parasites…). Je fais les commandes moi-même.
- Conventions Git : une branche par US (`feature/US-xx-slug`), Conventional Commits
  (`feat(ocr): …`, `fix`, `chore`, `docs`, `test`, `refactor`), PR vers `main` avec CI verte, merge après relecture.

## Objectif
- Projet **d'apprentissage et de portfolio** (Java, Spring Boot, PostgreSQL, Docker, OCR, Angular)
  ET outil perso pour FM27. En cas de conflit, l'apprentissage prime.
- Usage local, mono-utilisateur, **plusieurs sauvegardes FM** possibles.
- Backlog détaillé : `docs/BACKLOG.md`. Conditions de capture : `docs/FICHE_OCR.md`.

## Contexte
- Jeu : Football Manager 27 (sortie 10/11/2026), en **français**, sur un **PC Windows dédié au jeu**.
- Développement sur Windows et Ubuntu (les deux doivent fonctionner).
- Source de données : **OCR de la fiche joueur** (une capture PNG par joueur, lue avec Tesseract).
  Aucune modification du jeu. Pas d'export natif pour l'instant (importeur possible plus tard).
- Flux : captures manuelles sur le PC de jeu → dossier transféré → l'appli lit le dossier.
- Stack : Java 21, Maven (+ Maven Wrapper), **Spring Boot 4** (CLI via `CommandLineRunner`),
  Tess4J, java.awt / BufferedImage (prétraitement), Jackson 3 (package `tools.jackson`),
  **PostgreSQL dans Docker** (Docker Compose), Flyway, Spring Data JPA, JUnit 5, Testcontainers,
  Checkstyle (Google adapté à 4 espaces, `config/checkstyle/checkstyle.xml`), sortpom,
  GitHub Actions. Plus tard : Spring Web, Angular (dossier `frontend/`, même dépôt).
- Chemin `tessdata` dans `application.properties`, surchargeable par variable d'environnement.
- Lecture mémoire externe (JNA + Il2CppDumper) : idée de second projet, pas maintenant.

## Architecture (beans Spring injectés par constructeur)
- `model` : `Save` (sauvegarde), `Player`, `Snapshot` (relevé daté), `Attribute` (enum : libellé FR FM27
  + catégorie technique/mental/physique/gardien), plus tard `Role`, `Slot`.
- `io` : interface `PlayerImporter`, implémentation `OcrImporter` ; chargement des rôles (JSON).
  Le reste du code ne doit jamais savoir d'où viennent les données.
- `ocr` : prétraitement, gabarits (champ / gardien, JSON en pixels), découpe des zones,
  appel à Tesseract (config par type de zone), validation.
- `scoring` : calculs purs, aucune dépendance aux fichiers, à l'OCR ni à la base.
- `App` : orchestration uniquement.

## Décisions de données
- Modèle : Save 1→N Snapshot 1→N valeurs d'attributs N←1 Player.
- Identité joueur : id technique (PostgreSQL) + clé naturelle unique (sauvegarde, nom, date de naissance).
- Données lues en v1 : nom, date de naissance, postes (texte brut), pied préféré, attributs (entiers 1-20).
  Valeur, salaire, contrat, nationalité, taille : plus tard (v5).
- Import : `import --save "Lens" --game-date 2027-08-01 --dir captures/ [--replace]`.
  Date de jeu saisie à la main. Snapshot en double (même sauvegarde + date) refusé sauf `--replace`.
- Cellules douteuses : correction interactive dans la CLI, puis enregistrement en une transaction.
  Export JSON du snapshot en plus, pour le debug.
- Schéma géré par Flyway (pas de génération Hibernate).
- Rôles (v2) : données de référence dans `src/main/resources/roles.json`, pas en base.

## Feuille de route (ordre après la v1 ajustable selon le besoin)
**v1 — Extraction des attributs de mon effectif** (US-00 à US-11 du backlog)
O1. OCR brut d'une cellule découpée à la main
O2. Prétraitement + whitelist + PSM ; comparer avec O1
O3. Gabarits de la fiche (champ / gardien) en JSON + mode `calibrate`
O4. Une fiche → un `Player` (détection automatique du gabarit)
D1. Docker + PostgreSQL (Docker Compose)
D2. Flyway : schéma initial
D3. Spring Data JPA : enregistrer un Snapshot
D4. Testcontainers
O5. Validation + correction interactive des cellules douteuses
O6. Commande `import` complète via `PlayerImporter`
Critère de fin : ≥ 99 % de cellules correctes sur ~5 fiches de référence (JSON attendus écrits à la main).

**Après la v1**
- Capture automatique (`java.awt.Robot`, bouton « joueur suivant ») — appli installée sur le PC de jeu.
- v2 : score joueur × rôle (rôles FM27 relevés dans le jeu ; modèle avec/sans ballon à décider) + meilleur XI (algorithme hongrois).
- v3 : progression (comparaison de snapshots).
- v4 : API REST + front Angular.
- v5 : recrutement Moneyball (fiches hors effectif : plages "12-15", valeur, salaire).

## Modèle de score (v2, à confirmer)
- Attributs clés ×3, préférables ×2, moyenne ramenée sur 100.
- Attribut en plage "12-15" → milieu de la plage ; "-" → absent.

## Avancement
- Étape en cours : US-00 (mise en place), puis O1.
