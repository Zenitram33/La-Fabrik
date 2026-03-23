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

---

## PROJETS LA FABRIK

| Projet   | Statut      | Notes                              |
|----------|-------------|-------------------------------------|
| GymLog   | MVP terminé | V1 pre-store en cours               |
| _Fabrik  | En cours    | Skill PO créé · Agent Designer next |

---

## AGENT DESIGNER

### Déclencher l'Agent Designer entre MVP et V1
Le design ne se fait pas pendant le MVP — on code fonctionnel d'abord.
L'Agent Designer se déclenche une fois le MVP terminé et testé.
→ Skill : lafabrik-skill-designer.md
→ 2 questions de cadrage (thème + ambiance) puis tour des écrans
→ Maquettes générées dans Claude Chat avec le visualizer
→ Validation explicite avant de créer les sub-issues Linear
→ Sub-issues créées sous le ticket parent "Refonte design" (V1)

### Ne jamais mettre à jour Linear avant validation explicite
Claude a tendance à créer/modifier les tickets Linear trop tôt.
→ Toujours attendre la validation de Romain avant de toucher à Linear.
→ Finir le tour complet des écrans avant de créer les sub-issues.

### Maquettes visuelles dans Claude Chat
Claude peut générer des maquettes d'écrans mobile directement
dans Claude Chat avec le visualizer HTML. C'est suffisant pour valider
l'UX sans Figma ni outil externe.
→ 3 écrans max par rendu
→ Utiliser les CSS variables pour le dark/light mode
→ Simuler de vraies données (pas "Exercice 1")

---

## ORGANISATION LINEAR — COMPLÉMENTS

### Sub-issues pour les gros tickets
Un ticket de design ou de feature complexe doit être découpé
en sub-issues par écran ou par fonctionnalité.
→ Chaque sub-issue = un écran ou une feature atomique
→ Claude Code peut les traiter et les marquer Done indépendamment

### Priorité dans Linear — inutile pour un studio solo
La priorité High/Urgent/Medium n'apporte rien quand on est seul.
La numérotation dans le titre (1. 2. 3.) est suffisante et plus claire.
→ Laisser la priorité à None
→ Le numéro dans le titre = l'ordre de traitement

### Tickets La Fabrik — pas d'ordonnancement
Les tickets du projet La Fabrik (infra agents, skills) ne suivent
pas de séquence — ils sont traités selon la disponibilité.
→ Pas de numéro dans les titres La Fabrik
→ Pas de priorité

---

## PROJETS LA FABRIK

| Projet   | Statut        | Notes                                        |
|----------|---------------|----------------------------------------------|
| GymLog   | MVP terminé   | V1 pre-store en cours · design validé        |
| _Fabrik  | En cours      | Skill PO ✅ · Skill Designer ✅ · Linear configuré |

---

## AGENT TECH LEAD

### Déclencher à la fin de chaque milestone, pas de session
Trop fréquent à chaque session — pas assez de code accumulé.
→ Fin MVP → refacto → V1. Fin V1 → refacto → V2.

### Pas de ticket entrant — tickets créés en sortie
L'agent ne consomme pas de ticket Linear.
Il crée ses propres tickets Done pour chaque action effectuée.
→ Titre descriptif sans crochets : "Suppression imports inutilisés — stores"
→ Team Tech · labels [TechLead] + Refacto · statut Done
→ Traçabilité complète de la dette technique dans Linear

### Skill Claude Code uniquement
lafabrik-skill-techlead.md va dans skills/ du projet.
Claude Chat n'en a jamais besoin.

---

## VISION DISCOVERY AUTOMATISÉE — PROCHAIN PROJET

### Agents Discovery à construire
Pour le prochain projet, industrialiser la discovery avec ces agents :

**Agent PM**
→ Reçoit le pitch (2-3 min)
→ Pose les bonnes questions (méthode connue : Jobs-to-be-done, etc.)
→ Aide à définir les features MVP
→ Skill : chat/pm-discovery.md

**Agent PMM (Product Marketing Manager)**
→ Analyse de marché automatique
→ Recherche apps concurrentes (existantes, payantes, gratuites)
→ Identifie s'il y a une place à prendre
→ Génère un rapport de synthèse (PowerPoint ou HTML)
→ Skill : chat/pmm-market.md

### Pipeline d'automatisation Discovery → Delivery
```
Pitch → Agent PM → Agent PMM → Story map validée
→ Trigger GitHub (story map pushée)
→ Outil automation (Gumloop ou N8N)
→ Agent PO → tickets Linear créés
→ Notification Slack (ou autre) : "Linear prêt"
→ Claude Code consulte Linear → démarre le dev
```

### Notification
Prévoir un canal de notification (Slack ou autre) pour que
les agents puissent signaler la fin de leurs actions.
Ex : "Agent PO — 26 tickets créés dans GymLog V1"

### Choix outil automation
Gumloop vs N8N — à évaluer au moment de l'implémentation.
Gumloop : plus simple, pensé pour les agents IA
N8N : plus puissant, open source, self-hostable

### Rapport PMM
L'Agent PMM génère un livrable visuel (HTML ou PPTX ou Gamma)
avec l'analyse de marché — consultable par le porteur de projet.
