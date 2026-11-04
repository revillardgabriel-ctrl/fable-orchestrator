# Exécution des sous-agents par environnement

## 1. Claude Code / Cowork (Task tool)

Utiliser le Task tool avec le paramètre `model` explicite. Ne JAMAIS laisser le modèle par défaut.

```
Task(
  description="T1 extraction leads",
  prompt="<prompt sous-agent complet>",
  model="haiku"    # ou "sonnet"
)
```

- Lancer toutes les tâches parallélisables dans le MÊME tour (plusieurs Task calls d'un coup).
- Chaque Task = contexte vierge → le prompt doit être autonome (cf. prompts-sous-agents.md).
- Sortie machine-readable = demander du JSON et parser côté orchestrateur.

## 2. Claude.ai — Artifact "app orchestrateur" (API Anthropic)

Dans claude.ai, il n'y a pas de subagents natifs. Deux options :

**Option A (préférée pour les workflows répétés)** : créer un artifact React qui appelle l'API. Pas de clé à gérer, c'est déjà routé.

```javascript
async function runSubAgent(prompt, model = "claude-haiku-4-5-20251001") {
  const res = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model,                       // "claude-haiku-4-5-20251001" | "claude-sonnet-4-6"
      max_tokens: 1000,
      messages: [{ role: "user", content: prompt }]
    })
  });
  const data = await res.json();
  return data.content.filter(b => b.type === "text").map(b => b.text).join("\n");
}

// Parallélisation : Promise.all
const [t1, t2, t3] = await Promise.all([
  runSubAgent(promptT1),
  runSubAgent(promptT2),
  runSubAgent(promptT3, "claude-sonnet-4-6")
]);
```

- JSON attendu → nettoyer avant parse : `JSON.parse(text.replace(/```json|```/g, "").trim())`.
- Toujours try/catch par sous-agent : un échec ne doit pas tuer le batch.
- Afficher dans l'UI : statut par tâche (pending/running/done/failed), modèle utilisé, bouton retry.

**Option B (tâche one-shot, pas besoin d'app)** : l'orchestrateur exécute lui-même mais SIMULE la discipline d'orchestration — il rédige d'abord les prompts sous-agents comme livrables, puis les exécute séquentiellement en s'y tenant strictement. Moins bon en coût, mais garde la rigueur de décomposition. Le signaler à l'utilisateur.

## 3. n8n (multi-agents en workflow)

Architecture type :

```
Webhook/Trigger
  → [Nœud Anthropic - ORCHESTRATEUR (opus/fable)] : décompose, sort un JSON de sous-tâches
  → [Split In Batches / Item Lists] : 1 item par sous-tâche
  → [Switch sur champ "model"] 
      → [Nœud Anthropic - haiku]  (branche ouvriers)
      → [Nœud Anthropic - sonnet] (branche rédacteurs)
  → [Merge]
  → [Nœud Anthropic - ORCHESTRATEUR] : contrôle qualité + synthèse
  → Sortie (Sheets, Gmail, Firebase...)
```

Points clés :
- L'orchestrateur en tête produit un JSON strict : `[{"id": "T1", "model": "haiku", "prompt": "...", "depends_on": []}]`.
- Le Switch route sur `model`. Les branches tournent en parallèle nativement dans n8n.
- Contrôle qualité = 2e passage orchestrateur avec les sorties mergées ; boucle IF → retry vers la branche concernée (max 1 boucle, compteur dans les items).
- Sécurité : clé API en credential n8n, jamais en dur (cf. skill n8n-ai-agent-builder pour le hardening complet).

## 4. Bilan à afficher en fin d'orchestration (tous environnements)

```
BILAN ORCHESTRATION
Tâches : 6 (4 haiku, 1 sonnet, 1 orchestrateur)
Retries : 1 (T3, haiku→haiku, corrigé)
Estimation vs full-Opus : ~-70% de coût tokens
```

L'estimation est un ordre de grandeur (ratio de prix × volume estimé), pas une facture.
