# LA FABRIK · SKILL — LINEAR DEV
# =============================================================
# Skill pour Claude Code
# Permet de lire les tickets Linear et mettre à jour leur statut
# Utilisé pendant les sessions de développement
# =============================================================

## RÔLE
Quand tu démarres une session de développement, tu consultes Linear
pour savoir quoi faire. Tu ne lis pas la story map — Linear est la
source de vérité unique.

---

## DÉMARRAGE DE SESSION

### 1. Identifier le milestone actif
```
- Si MVP non terminé → travailler les tickets MVP Backlog par ordre
- Si MVP Done → travailler les tickets V1 pre-store par ordre (1, 2, 3...)
- Si V1 Done → travailler les tickets V2 par ordre
```

### 2. Lister les prochains tickets
Via l'API Linear, récupérer les tickets Backlog du milestone actif,
triés par priorité (Urgent → High → Medium → Low) puis par numéro dans le titre.

### 3. Proposer avant d'agir
```
"Voici les prochains tickets à traiter :
- TECH-4 : Session Refacto (Urgent)
- PRD-19 : Refonte design (Urgent)
- TECH-3 : EAS Build → TestFlight (Urgent)

Je commence par TECH-4. C'est bon ?"
```
Attendre la validation avant de démarrer.

---

## WORKFLOW PAR TICKET

Pour chaque ticket, dans cet ordre strict :

### 1. Lire le ticket
Récupérer depuis Linear :
- Titre
- Description complète
- Critères d'acceptance
- Estimate (points)
- Labels (type de ticket)

### 2. Passer le ticket "In Progress"
Mettre à jour le statut Linear : Backlog → In Progress

### 3. TDD
- Écrire les tests selon les critères d'acceptance
- Vérifier qu'ils FAIL
- Commiter : `test: {ticket-id} — red`

### 4. Implémenter
- Développer la fonctionnalité
- Vérifier que les tests passent
- Commiter : `feat: {ticket-id} — {titre}`

### 5. Générer la recette end-to-end
Produire la liste des scénarios à tester manuellement,
dérivés des critères d'acceptance du ticket.
Format :
```
RECETTE — {titre du ticket}
─────────────────────────────
1. {action} → {résultat attendu}
2. {action} → {résultat attendu}
3. {action} → {résultat attendu}
```

### 6. S'arrêter et attendre
NE PAS marquer Done automatiquement.
Présenter la recette à Romain et attendre sa validation.

### 7. Sur validation de Romain
- Marquer le ticket Done dans Linear
- Mettre à jour PROGRESS.md
- Commiter : `chore: update PROGRESS.md — {ticket-id} done`
- Demander : "Ticket {id} terminé ✅. On passe au suivant ?"
- Attendre la réponse avant de continuer

---

## API LINEAR — OPÉRATIONS COURANTES

### Lire un ticket
```
GET /issues/{ticket-id}
Retourne : titre, description, critères, statut, labels, estimate
```

### Passer In Progress
```
PATCH /issues/{ticket-id}
{ "stateId": "<id-state-in-progress>" }
```

### Marquer Done
```
PATCH /issues/{ticket-id}
{ "stateId": "<id-state-done>" }
```

### Créer un ticket Bug (si problème découvert)
```
POST /issues
{
  "title": "Bug: {description courte}",
  "description": "## Contexte\n...\n## Étapes pour reproduire\n...\n## Comportement attendu\n...",
  "labels": ["QA", "Bug"],
  "priority": 1,
  "projectId": "<gymlog-project-id>",
  "milestoneId": "<milestone-actif-id>"
}
```

---

## RÈGLES

- Ne jamais marquer Done sans validation explicite de Romain
- Ne jamais passer au ticket suivant sans demande explicite
- Si un ticket bloque → créer un ticket Bug + signaler à Romain
- Toujours lire le ticket complet avant de coder
- Les critères d'acceptance du ticket = les tests à écrire

---

## COMMANDE D'AMORCE
Pour démarrer une session avec ce skill :
```
Lis CLAUDE.md et lafabrik-skill-linear-dev.md.
Consulte Linear, identifie le prochain ticket à traiter
et propose-le moi avant de commencer.
```

Pour traiter un ticket spécifique :
```
Lis CLAUDE.md et lafabrik-skill-linear-dev.md.
Traite le ticket {ticket-id}.
```
