# Routage détaillé des sous-tâches

## Table de routage par type de tâche

| Type de tâche | Modèle | Justification |
|---|---|---|
| Extraction de données (CSV, HTML, PDF → JSON) | haiku-4-5 | Déterministe, format strict suffit |
| Classification / scoring (leads, sentiments, priorités) | haiku-4-5 | Critères explicites dans le prompt |
| Recherche web factuelle (1 question = 1 réponse) | haiku-4-5 | Le tool fait le travail |
| Traduction, reformatage, nettoyage de texte | haiku-4-5 | Mécanique |
| Résumé de document unique (< 20 pages) | haiku-4-5 | Compression simple |
| Génération en série depuis template (emails, fiches produit, variantes A/B) | haiku-4-5 | Le template porte l'intelligence |
| Rédaction longue originale (article, rapport, LM) | sonnet-4-6 | Cohérence + style |
| Code applicatif (composant, fonction, script) | sonnet-4-6 | Raisonnement structurel |
| Analyse croisée multi-documents | sonnet-4-6 | Mise en relation |
| Raisonnement métier (stratégie commerciale, pricing, argu de vente) | sonnet-4-6 | Nuance |
| Debug complexe, architecture, arbitrage entre approches | orchestrateur | Trop de contexte implicite |
| Rédaction des prompts des sous-agents | orchestrateur | C'est le cœur de sa valeur |

## Règle de décision rapide

1. La tâche a-t-elle un format d'entrée ET de sortie parfaitement définissables ? → **Haiku**.
2. La tâche demande-t-elle du style, de la cohérence longue ou du raisonnement en plusieurs étapes ? → **Sonnet**.
3. La tâche exige-t-elle de comprendre l'ensemble du projet / de faire des arbitrages ? → **Orchestrateur** (et se demander si elle est vraiment nécessaire).

En cas de doute entre deux niveaux : prendre le niveau inférieur, avec critères de sortie stricts. L'escalade coûte moins cher qu'un sur-dimensionnement systématique.

## Ordres de grandeur de coût (tokens input/output, base API 2026 — vérifier si critique)

- Haiku 4.5 ≈ $1 / $5 par Mtok
- Sonnet 4.6 ≈ $3 / $15 par Mtok
- Opus/Fable ≈ $15 / $75 par Mtok

Conséquence : 10 sous-tâches Haiku coûtent moins cher qu'une seule exécution Opus équivalente. La parallélisation est gratuite en coût, elle ne coûte que du temps orchestrateur.

## Escalade

- Échec Haiku après 1 retry avec feedback → Sonnet, même prompt enrichi du feedback.
- Échec Sonnet après 1 retry → l'orchestrateur reprend la tâche LUI-MÊME (dernier recours) et note la cause dans le bilan final.
- Jamais plus de 2 niveaux d'escalade sur une même sous-tâche.

## Cas limites

- **Tâche unique mais énorme** (ex : résumer 300 pages) : découper en chunks → N sous-agents Haiku en parallèle → 1 sous-agent Sonnet pour la fusion → orchestrateur valide.
- **Chaîne séquentielle stricte** (T2 a besoin de la sortie de T1) : conserver le même modèle sur toute la chaîne si les tâches sont homogènes, pour limiter les ruptures de style.
- **Sortie destinée à une machine** (JSON pour Firebase, n8n, Sheets) : toujours Haiku + schéma JSON dans le prompt + instruction "réponds UNIQUEMENT en JSON, sans backticks".
