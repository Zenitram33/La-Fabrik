# LA FABRIK OS — VISION & ROADMAP
# =============================================================
# La Fabrik = une usine à produits digitaux automatisée
# Objectif : MVP en une semaine sur n'importe quelle idée
# Stack cible : Claude API + N8N + GitHub + Linear + Slack
# =============================================================

## VISION

Simuler le fonctionnement d'une équipe produit complète via des agents IA
orchestrés, avec validation humaine à chaque étape clé.

Court terme  → Outil interne pour Romain (MVP en 1 semaine)
Moyen terme  → Produit : SaaS ou open source + service
Long terme   → Plateforme accessible à des non-développeurs

## PÉRIMÈTRE ACTUEL
- Apps mobiles iOS + Android (Expo + React Native + Supabase)
- À venir : sites web, apps full-stack

---

## LES AGENTS — INVENTAIRE COMPLET

### PHASE 1 — DISCOVERY

**Agent Discovery** (Claude Chat — semi-automatisé)
- Rôle : reformuler l'idée, scoring, epics, stories, data model, stack
- Trigger : pitch humain dans Claude Chat
- Output : story map HTML + brief + data model SQL → poussés sur GitHub
- Statut : ✅ Skill partiel (7 étapes)
- Note : validation humaine obligatoire à chaque étape

**Agent PMM** (Claude API + web search — automatisé)
- Rôle : étude marché (concurrents, fonctionnalités, monétisation, entrée marché)
  → Si marché saturé : signaler + proposer alternatives ou pivots
  → Les fonctionnalités MVP du brief sont dans l'input pour comparer
- Trigger : brief poussé sur GitHub dans le nouveau repo projet
- Output : rapport lisible (HTML ou présentation) + recommandations
  → Format : à définir (Gamma ou équivalent gratuit)
- Statut : ❌ À créer

**Agent Identité** (Claude Chat — semi-manuel)
- Regroupe Naming + Logo en un seul skill
- Sous-skill Naming : brainstorming nom, vérification dispo App Store
- Sous-skill Logo : génération brief visuel + prompt pour ChatGPT/DALL-E
  → Claude ne génère pas le logo — il génère le prompt et valide avec Romain
- Trigger : brief validé
- Output : nom validé + icon.png 1024x1024 poussé sur GitHub
- Statut : ❌ À créer

---

### PHASE 2 — DELIVERY

**Agent PO** (Claude API — automatisé)
- Rôle : lire story map sur GitHub → créer tous les tickets Linear
- Trigger : story map validée pushée sur GitHub
  → N8N détecte le fichier → appelle Claude API → Linear API
- Output : tickets Linear (milestones, labels, estimates)
- Statut : ✅ Skill complet (Claude Chat) → à porter en automatisé N8N

**Agent Designer** (Claude Chat — semi-automatisé)
- Rôle : définir UX/design system écran par écran avec Romain
- Trigger : MVP terminé — entre MVP et V1
- Output : issues design dans Linear (issues directes, pas sub-issues)
- Statut : ✅ Skill complet
- Note : pertinent de garder en V1 — évite les refactos (leçon GymLog timer)

**Agent Dev** (Claude Code — manuel déclenché)
- Rôle : implémenter les tickets milestone par milestone
  → Lire Linear au démarrage, traiter ticket par ticket
  → TDD, bonne architecture, sécurité, conventions La Fabrik
- Trigger : manuel — Romain lance Claude Code
  → Pas automatisable via N8N (Claude Code ≠ API simple)
- Transition vers automatisation :
  → Claude Code lit Linear au démarrage et liste les tickets du milestone
  → Suggère lui-même de lancer Agent TechLead en fin de milestone
- Output : code committé sur GitHub
- Statut : ⚠️ Skill Agent Dev générique à créer

**Agent TechLead** (Claude Code — manuel déclenché)
- Rôle : audit code, suppression code mort, refacto, dette technique
- Trigger : manuel — proposé par Agent Dev en fin de milestone
- Output : commit refacto sur GitHub
- Statut : ⚠️ Skill dédié à créer

---

### PHASE 3 — RELEASE & QA

**Agent QA** (Claude Chat — semi-manuel)
- Rôle : guider recette manuelle, générer RECETTE-SESSION-N.md, tickets bugs
- Trigger : build preview disponible
- Output : fichier recette + bugs dans Linear
- Statut : ❌ À créer
- Vision long terme : tests E2E automatisés → gate avant build si tests OK

**Agent DevOps** — 3 sous-skills :

  1. Setup initial (Claude Chat — guidé, une fois par projet)
     → App ID, App Store Connect, Supabase auth, EAS config
     → Statut : ✅ Skill DevOps Release Setup

  2. Build & TestFlight (automatisable)
     → Trigger : validation recette QA
     → eas build → notif Slack "Build preview disponible"
     → Statut : ❌ À automatiser

  3. Soumission stores (semi-automatisé)
     → eas submit iOS + Android + fiche App Store
     → Statut : ❌ À créer

---

## ORCHESTRATION — FLOW CIBLE

```
Humain pitche l'idée
        ↓
[SEMI-AUTO] Agent Discovery → 7 étapes validées
        ↓  → brief + story map pushés sur GitHub
[AUTO] N8N trigger → Agent PMM → rapport marché
        ↓  → notif Slack : "Rapport PMM prêt"
[HUMAIN] Validation go/no-go + pivot éventuel
        ↓
[SEMI-AUTO] Agent Identité → naming + logo
        ↓
[AUTO] N8N trigger → Agent PO → tickets Linear
        ↓  → notif Slack : "X tickets créés"
[HUMAIN] Review tickets
        ↓
[SEMI-AUTO] Agent Designer → UX validé (avant V1)
        ↓
[MANUEL] Agent Dev (Claude Code) → milestone par milestone
        ↓  → suggère Agent TechLead en fin de milestone
[MANUEL] Agent TechLead → refacto → push GitHub
        ↓
[SEMI-AUTO] Agent QA → recette guidée
        ↓  → si OK : trigger build
[AUTO] Agent DevOps Build → eas build → notif Slack
        ↓
[SEMI-AUTO] Agent DevOps Submit → soumission stores
        ↓  → notif Slack : "App soumise"
```

## NIVEAUX D'AUTOMATISATION

```
FULL AUTO    → Agent PMM, Agent PO, notifications Slack, builds
SEMI-AUTO    → Discovery, Designer, QA, Submit
GUIDÉ        → DevOps Setup
MANUEL       → Agent Dev, Agent TechLead (Claude Code)
TRANSITION   → Agent Dev/TechLead proposent eux-mêmes l'étape suivante
```

---

## STACK TECHNIQUE

Orchestration : N8N (open source, self-hosted gratuit)
→ Triggers : GitHub webhooks, Linear webhooks
→ Actions : Claude API, Slack, GitHub API, Linear API

Notifications : Slack
→ #lafabrik-notifications → toutes les notifs
→ #lafabrik-{projet} → notifs par projet

Stockage : GitHub
→ Repo lafabrik → skills + rétro + vision
→ Repo {projet} → code + discovery + recettes

Tests :
→ Court terme : TDD (stores Zustand, utils, queries Supabase)
→ Moyen terme : tests E2E → gate avant build automatique
→ Long terme : non-régression → déclenche Agent DevOps si OK

---

## ROADMAP — OBJECTIF FIN AVRIL

### Semaine 1
- [ ] Recette GymLog + soumission App Store
- [ ] Skill Agent Dev générique (TDD + archi + sécurité)
- [ ] Skill Agent TechLead
- [ ] Skill Agent QA

### Semaine 2
- [ ] Skill Agent PMM
- [ ] Skill Agent Identité (Naming + Logo)
- [ ] Setup N8N
- [ ] Automatiser : brief GitHub → Agent PMM → notif Slack

### Semaine 3
- [ ] Automatiser : story map GitHub → Agent PO → Linear
- [ ] Automatiser : build preview → notif Slack
- [ ] 2ème projet : test du flow complet

### Semaine 4
- [ ] Itérations retours 2ème projet
- [ ] V1 La Fabrik OS opérationnelle
- [ ] Documentation flow complet

---

## DÉCLINAISONS FUTURES

| Type projet    | Ce qui change                        |
|----------------|--------------------------------------|
| Site web       | Expo/RN → Next.js, même Supabase     |
| Android        | Même process EAS + Google Play       |
| App full-stack | Agent Backend si API custom          |
| SaaS           | Agent Stripe/monétisation            |

## CE QUI NE CHANGE JAMAIS
- Process discovery 7 étapes
- Linear comme source de vérité
- GitHub comme dépôt
- Supabase comme backend par défaut
- Validation humaine aux étapes clés
