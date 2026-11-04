# Fable Orchestrator : un skill pour Claude

*Par [Onari agency](https://onari-agenct.com), l'agence web et IA de Lyon et Genève (TikTok : [@onari_agency](https://www.tiktok.com/@onari_agency)).*

Tu trouves que Claude dévore tes tokens ? Ce skill change sa façon de travailler. Le gros modèle ne fait
plus tout lui-même : il devient **chef d'orchestre**.

1. Il comprend ta demande et la découpe en petites tâches.
2. Il écrit un **brief** clair pour chaque tâche : l'objectif, les fichiers, le format attendu.
3. Il fait exécuter ces tâches par des modèles plus légers et moins chers (Haiku, Sonnet), en parallèle.
4. Il relit tout, fait corriger ce qui ne va pas, puis te livre le résultat.

Tu gardes la même qualité et tu dépenses beaucoup moins de tokens.

## Installer

**Sur claude.ai ou l'appli Claude**
1. Télécharge [`fable-orchestrator.zip`](fable-orchestrator.zip) (bouton « Download raw file »).
2. Va dans Réglages → Capacités → Skills.
3. Importe le fichier .zip.

**Sur Claude Code**
```bash
git clone https://github.com/revillardgabriel-ctrl/fable-orchestrator.git
cp -r fable-orchestrator/fable-orchestrator ~/.claude/skills/
```

## L'utiliser

Sur une grosse tâche, dis simplement à Claude **« orchestre »**. Le skill se déclenche aussi tout seul
dès qu'une tâche peut se découper en plusieurs morceaux indépendants.

Pour une petite question, il ne s'active pas : découper ne ferait que perdre du temps.

## Contenu

- [`fable-orchestrator/SKILL.md`](fable-orchestrator/SKILL.md) : la méthode en 5 phases.
- [`references/routing.md`](fable-orchestrator/references/routing.md) : quel modèle pour quelle tâche.
- [`references/prompts-sous-agents.md`](fable-orchestrator/references/prompts-sous-agents.md) : des modèles de briefs.
- [`references/execution.md`](fable-orchestrator/references/execution.md) : la mise en place sur Claude Code, en artifact ou avec n8n.

---

Tu veux de l'IA qui travaille vraiment dans ton entreprise ? Écris-nous sur [onari-agenct.com](https://onari-agenct.com).
