# LA FABRIK · RETROSPECTIVE
# =============================================================
# Mémoire globale inter-projets — à donner en contexte à Claude
# au démarrage de chaque nouveau projet ou session de travail.
#
# Contient uniquement les apprentissages réutilisables.
# Les retours spécifiques à un projet restent dans ce projet.
#
# Usage : "Lis STUDIO-RETROSPECTIVE.md avant de commencer."
# =============================================================

## PROCESS DISCOVERY

### Ne pas générer le livrable final avant validation de toutes les étapes
Claude a tendance à vouloir générer la story map finale trop tôt.
→ Respecter strictement les 7 étapes dans l'ordre.
→ Aucun livrable de synthèse avant validation de toutes les étapes précédentes.

### Challenger la notion de "programme" ou d'entité centrale
Pour chaque app, la question "qu'est-ce que l'entité principale ?" est critique
pour le data model. La poser explicitement avant de valider les epics.

### Proposer la fusion des epics qui partagent le même écran
Si deux epics touchent le même front et les mêmes données, proposer la fusion.
→ Moins de complexité, plus de cohérence dans l'implémentation.

### Faire la Discovery avant chaque milestone
Le timer V1 a été implémenté sans discovery → solution rejetée par Romain.
→ Bloquer une session Discovery avant de créer les tickets de chaque milestone.
→ Aucun ticket de feature sans story validée.
→ Le coût d'un refacto non planifié dépasse largement celui d'une discovery.

---

## SESSION 0 — SETUP

### Toujours inclure GitHub dans la Session 0
→ git init
→ git remote add origin https://github.com/{compte}/{projet}
→ git add . && git commit -m "chore: init project — session 0"
→ git push -u origin main

### Toujours inclure Expo EAS dans la Session 0
→ npx expo login
→ eas init --id com.lafabrik.{nom-projet}
→ Configurer app.json avec slug et owner

### Toujours mentionner le tunnel Expo pour tests en mobilité
→ npx expo start --tunnel
→ Utile pour tester hors domicile sans dépendance au réseau local

---

## TDD

### Préciser le périmètre des tests explicitement
Sans contrainte, Claude Code teste tout — composants, gestes, navigation.
→ Toujours préciser dans CLAUDE.md :
  TESTER : stores Zustand, fonctions utilitaires, queries Supabase
  NE PAS TESTER : composants UI, gestes, navigation, librairies tierces

---

## SÉCURITÉ

### Supabase : ne pas activer Automatic RLS
L'option génère des policies génériques qui conflictent avec les policies custom.
→ Toujours désactiver Automatic RLS.
→ Toujours activer Enable Data API.
→ Nos policies sont écrites à la main dans le fichier SQL.

---

## DATA MODEL

### Stocker les données dans l'unité de base internationale
→ Poids en kg, distances en mètres, températures en Celsius.
→ Convertir côté client selon les préférences utilisateur.
→ Évite les bugs si l'utilisateur change d'unité.

### Challenger les tables intermédiaires
Les tables de jointure sont souvent inutiles si on peut calculer côté client.
→ Supprimer si l'indicateur peut se calculer depuis les données existantes.

---

## FICHIERS DE RÉFÉRENCE

### CLAUDE.md doit rester court (~60 lignes max)
Lu entièrement à chaque démarrage — consomme du contexte inutilement si trop long.
→ CLAUDE.md = règles permanentes + pointeurs vers fichiers détaillés.
→ Fichiers détaillés lus à la demande selon le contexte.

### PROGRESS.md mis à jour à chaque commit
→ Convention : `chore: update PROGRESS.md — S{X.Y} done`
→ Pas à la fin de session — trop flou comme déclencheur.

### Séparer data model et seed en deux fichiers SQL
→ {projet}-data-model.sql : schéma + RLS + indexes
→ {projet}-seed.sql : données initiales
→ Permet de rejouer le seed sans retoucher le schéma.

### Ajouter une recette manuelle par session
→ Fichier RECETTE-SESSION-{N}.md généré après chaque session.
→ Format : scénario / action / résultat attendu / ✅ ❌
→ Dérivé directement des critères d'acceptance des stories.

---

## CLAUDE CODE — PERMISSIONS

### Définir les commandes auto-approuvées dans CLAUDE.md
→ AUTO-APPROUVÉES (sans confirmation) :
  cd, ls, cat, pwd, echo, mkdir, cp, mv
  git status, git log, git diff, git add
  npm test, npm run, npx expo start
  supabase status, supabase db diff
→ CONFIRMATION OBLIGATOIRE :
  git push, git commit, supabase db push, supabase db reset, rm

---

## QUALITÉ CODE

### Session Refacto entre chaque lot
→ Avant chaque nouveau lot, lancer une session Claude Code dédiée :
  "Tu es tech lead. Audite le code, supprime le code mort,
   factorise les duplications. Ne touche pas aux fonctionnalités."
→ Checklist : imports inutilisés, composants morts, types any,
  console.log de debug, dépendances npm inutilisées (npm prune)

---

## INFRASTRUCTURE LA FABRIK

### Versionner les skills sur GitHub
→ Repo : `lafabrik` (skills + rétro)
→ CLAUDE.md pointe vers les URLs raw GitHub
→ Une amélioration = un commit = tous les projets en bénéficient

### Structure standard d'un projet
```
{projet}/
├── discovery/         ← story map, data model, seed, sessions, brief
├── skills/            ← skills Claude Code spécifiques au projet
├── CLAUDE.md          ← pointe vers lafabrik sur GitHub
├── PROGRESS.md
└── RECETTE-SESSION-{N}.md
```

### Séparer skills Claude Chat et Claude Code
→ Claude Chat : Agent PO, Agent Designer (discovery)
→ Claude Code : Agent Dev, Agent TechLead, Agent QA (delivery)
→ Le dossier skills/ dans Claude Code = uniquement skills delivery

---

## ORGANISATION LINEAR

### Structure workspace La Fabrik
→ Teams : Product · Dev
→ Projets : un par app + projet La Fabrik (infra agents)
→ Milestones dans les projets : MVP · V1 pre-store · V2
→ Labels groupe Agent : [PO] [Designer] [Dev] [QA] [TechLead]
→ Labels groupe Type : Feature · Design · Bug · Infra · Refacto
→ Toujours deux labels par ticket (1 Agent + 1 Type)
→ Estimation Fibonacci natif Linear : 1 · 2 · 3 · 5 · 8

### Token Linear — même token pour Claude Chat et Claude Code
→ Le MCP Linear (Claude Chat) et l'API Linear (Claude Code) utilisent
  le même Personal API Key dans Linear Settings.
→ Quand le token expire, les deux tombent simultanément.
→ Fix unique : Linear Settings → API → Personal API keys → régénérer
  puis mettre à jour dans .env.local (Claude Code) ET reconnecter
  le MCP dans claude.ai Settings → Connections.

---

## NAMING & BRANDING

### Le nom App Store n'est pas définitif
→ Bundle ID = permanent et invisible des users. Ne jamais changer.
→ Nom affiché sur l'App Store = modifiable à tout moment dans App Store Connect.
→ Ne pas bloquer une V1 ou une mise en prod sur le naming.
→ Industrialiser le brainstorming naming dans un skill dédié (à créer).

### "GymLog" tout attaché est déjà pris sur l'App Store
→ Deux apps existantes : "GymLog App" (GymLog Inc.) et "GymLog: Workout Tracker".
→ App Store Connect bloquera l'enregistrement du nom exact "GymLog".
→ Noms disponibles probables : Sett, Reppr, Forj, Liftr (à vérifier au moment de la soumission).

---

## APPLE / TESTFLIGHT

### Deux consoles distinctes — rôles différents
→ developer.apple.com = console technique (App IDs, certificates, provisioning profiles,
  capabilities). EAS gère souvent cette partie automatiquement.
→ App Store Connect = console distribution (fiche app, builds, TestFlight, soumission review).
  C'est la console principale pour la mise en prod.

### Apple Sign-In : tester sur build EAS, pas Expo Go
→ Dans Expo Go, Apple Sign-In crée un compte lié au bundle ID d'Expo Go,
  pas à com.lafabrik.{projet}. Les comptes créés sont "parasites" et sans impact prod.
→ Recette Apple Sign-In = uniquement sur build EAS (TestFlight ou dev build).

---

## PROJETS LA FABRIK

| Projet   | Statut           | Notes                                        |
|----------|------------------|----------------------------------------------|
| GymLog   | V1 pre-store ✅  | TestFlight + images exercices next           |
| _Fabrik  | En cours         | Skill PO créé · Skill Designer créé          |
