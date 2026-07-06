Voici la vérification, critère par critère, avec les fichiers qui prouvent (ou nuancent) chaque point.

## 1. Routage instantané via FAISS — ✅ conforme
[src/backend/tools/faiss_tool.py](../src/backend/tools/faiss_tool.py) : `validate_film()` est le **premier** outil appelé (voir [tools_registry.py](../src/backend/agent/tools_registry.py) — `lookup_movie`, `find_similar`, `movie_age` appellent tous `faiss_tool.validate_film` avant tout accès SQL). Index chargé en mémoire (singleton), recherche par similarité cosinus avec seuil (`faiss_score_threshold = 0.75` dans [config.py:81](../src/backend/config.py)). En dessous du seuil → `None` (film rejeté, pas de coût SQL/scraping). Testé dans [tests/test_faiss_tool.py](../tests/test_faiss_tool.py).

## 2. Pas de gel Streamlit grâce à l'asynchronisme FastAPI — ✅ conforme
Toutes les routes sont `async` ([routes.py](../src/backend/api/routes.py)). Le graphe LangGraph (bloquant car Ollama + tools sont synchrones) est exécuté via `asyncio.to_thread` dans [graph.py:136](../src/backend/agent/graph.py#L136) avec un commentaire explicite « **NE PAS geler la boucle d'evenements** ». Le front appelle l'API en HTTP avec un `timeout=60` ([api_client.py:21](../src/front/common/api_client.py#L21)) — le calcul lourd ne bloque jamais l'event loop de l'API, donc pas de gel global côté serveur.

## 3. 0% hallucination sur les métadonnées de base — ✅ conforme avec une nuance
Le prompt système impose d'utiliser **uniquement** les données de `lookup_movie` ([prompts.py:12-14](../src/backend/agent/prompts.py#L12-L14)), et le nœud Juge audite la fidélité de chaque réponse ([judge.py](../src/backend/agent/judge.py)).
⚠️ Nuance : le juge est **fail-open** (`judge.py:71` — si le LLM juge plante, on `return True` = validé quand même). C'est un choix assumé (ne pas casser l'agent), mais ça veut dire que la garantie 0% hallucination repose sur le prompt de l'agent seul dans ce cas précis, pas sur un filet de sécurité déterministe.

## 4. Gestion des limites de connaissance (absent base + absent Wikipédia) — ✅ conforme, mais reste du soft-prompting
- `lookup_movie` renvoie `{"found": false}` si absent ([tools_registry.py:29-31](../src/backend/agent/tools_registry.py#L29-L31)), et le prompt impose « dis poliment que tu ne connais pas ce film » (règle 1) et « si vraiment introuvable, dis que tu ne sais pas » (règle 7).
- `wikipedia_synopsis` ne renvoie jamais `null`, mais un message explicite du type *« Aucune page Wikipedia trouvée pour X »* ([wikipedia_tool.py:34](../src/backend/tools/wikipedia_tool.py#L34)).
⚠️ Il n'y a pas de règle **codée en dur** qui empêche l'invention si les deux sources échouent — c'est le prompt + le Juge qui portent cette garantie, pas une logique déterministe. Pour un petit modèle local (qwen2.5:7b), le risque résiduel n'est donc pas nul à 100%, mais l'architecture (prompt strict + juge + fallback borné) est bien celle attendue par le critère.

## 5. Web-scraping strictement sélectif — ⚠️ conforme en intention, mais non appliqué en dur
Le déclenchement de `wikipedia_synopsis` est **uniquement** piloté par le prompt : règle 4 dans [prompts.py:18-19](../src/backend/agent/prompts.py#L18-L19) (« N'utilise `wikipedia_synopsis` QUE si... ») et le docstring du tool lui-même ([tools_registry.py:82-84](../src/backend/agent/tools_registry.py#L82-L84)) répète la consigne pour influencer la décision du LLM.
C'est donc un contrôle **soft** (le LLM décide), pas un gate côté code qui vérifierait par exemple que `lookup_movie` a été appelé et que sa réponse est jugée insuffisante avant d'autoriser l'appel Wikipedia. Ça respecte l'esprit du critère mais pas une garantie stricte au sens algorithmique.

## 6. Aucune requête SQL brute générée par le LLM — ✅ conforme, garantie forte
Le LLM ne voit **jamais** de SQL : il n'a accès qu'à des tools LangChain typés (`title: str`, `k: int`) via [tools_registry.py](../src/backend/agent/tools_registry.py). Toutes les requêtes vivent dans [src/backend/data/repository.py](../src/backend/data/repository.py), en SQL **paramétré** (`text(...)` + dict de binds, jamais de f-string/concat) — donc pas d'injection possible. Le docstring du fichier l'affirme explicitement : *« REGLE D'OR (brief) : le LLM ne genere JAMAIS de SQL »*. C'est le critère le mieux garanti architecturalement (pas juste par prompt).

---

### Résumé
| # | Critère | Verdict |
|---|---|---|
| 1 | Routage FAISS instantané | ✅ garanti par le code |
| 2 | Pas de gel Streamlit (async FastAPI) | ✅ garanti par le code |
| 3 | 0% hallucination métadonnées | ✅ mais juge fail-open (pas 100% déterministe) |
| 4 | Limites de connaissance maîtrisées | ✅ mais repose sur le prompt + juge, pas une règle dure |
| 5 | Scraping strictement sélectif | ⚠️ soft (décision du LLM), pas de gate code |
| 6 | Aucun SQL brut du LLM | ✅ garanti architecturalement (le plus solide) |

Rien n'est cassé ni absent, mais si un jury exigeant veut des garanties **déterministes** sur les points 3, 4 et 5, ce sont actuellement des garanties **de prompt engineering + juge LLM**, pas des invariants de code.
