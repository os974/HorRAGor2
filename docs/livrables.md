# État des livrables

## 1. Code source applicatif — ✅ présent

Dépôt Git structuré à la racine du projet.

- **Front Streamlit** : [src/front/streamlit/app.py](../src/front/streamlit/app.py) (+ client HTTP dans [src/front/common/api_client.py](../src/front/common/api_client.py))
- **API FastAPI** : [src/backend/api/main.py](../src/backend/api/main.py), [routes.py](../src/backend/api/routes.py), [deps.py](../src/backend/api/deps.py)
- **Moteur LangGraph** : [src/backend/agent/graph.py](../src/backend/agent/graph.py) (agent ReAct), [judge.py](../src/backend/agent/judge.py) (juge LLM), [tools_registry.py](../src/backend/agent/tools_registry.py), [prompts.py](../src/backend/agent/prompts.py), [state.py](../src/backend/agent/state.py)
- **Outils data** : [src/backend/tools/](../src/backend/tools/), [src/backend/data/](../src/backend/data/)
- **Tests** : [tests/](../tests/) (18 modules pytest couvrant agent, API, tools, contrats, données)

Rien à faire ici.

## 2. Configuration industrielle (`pyproject.toml` + `uv`) — ✅ présent

[pyproject.toml](../pyproject.toml) à la racine :

- dépendances applicatives déclarées (FastAPI, Streamlit, LangGraph, FAISS, pgvector, SQLAlchemy, langchain-ollama…)
- `[project.optional-dependencies].dev` avec ruff + pytest
- 9 scripts `[project.scripts]` exposés (`horragor-api`, `horragor-graph`, `horragor-ingest`, etc.)
- config `ruff` (lint) et `pytest` (asyncio, pythonpath)
- lockfile [uv.lock](../uv.lock) présent et à jour, `.venv` généré par `uv`

Rien à faire ici.

## 3. Schéma du graphe (Mermaid) — ✅ présent, deux variantes

- [docs/graphe_agent.mmd](graphe_agent.mmd) — version **manuelle/pédagogique** (stateDiagram-v2), avec les étapes Agent → Tools → Juge → Verdict → Fallback commentées.
- [docs/graphe_agent_genere.mmd](graphe_agent_genere.mmd) — version **générée depuis le code réel** via `uv run horragor-graph` (commande exposée par [export_graph.py](../src/backend/agent/export_graph.py)), régénérable à tout moment.
- [docs/architecture_globale.mmd](architecture_globale.mmd) — schéma d'architecture globale (front/API/agent/outils/BDD), complémentaire au graphe de l'agent.

## 4. Support de pitch (1 à 2 slides max) — ⚠️ existe mais ne respecte pas le format demandé

[docs/prez/pitch.md](prez/pitch.md) est un deck **Marp** de **10 slides** (~10 min de présentation), pas 1-2 slides. Le [README.md](prez/README.md) du dossier le signale déjà explicitement : le brief demande 1-2 slides max, et propose que les slides 1-2 du deck actuel servent de « pitch d'accroche » autonome si besoin.

À trancher avant la remise :
- soit condenser le deck en un support 1-2 slides dédié (nouveau fichier, ex. `docs/prez/pitch-court.md`),
- soit confirmer avec le jury/référent que le deck long est accepté en plus/à la place.

Pas d'export PDF/PPTX trouvé dans le dépôt — à générer via Marp (`npx @marp-team/marp-cli`) au moment de la remise si un fichier binaire est exigé.

## Résumé

| Livrable | Statut | Emplacement |
|---|---|---|
| Code source (Front/API/LangGraph) | ✅ OK | `src/`, `tests/` |
| `pyproject.toml` + `uv` | ✅ OK | `pyproject.toml`, `uv.lock` |
| Schéma du graphe Mermaid | ✅ OK (à régénérer si le code a bougé) | `docs/graphe_agent*.mmd`, `docs/architecture_globale.mmd` |
| Support de pitch (1-2 slides) | (deck de 10 slides actuellement) | `docs/prez/pitch.md` |
