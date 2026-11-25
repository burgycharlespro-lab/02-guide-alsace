# Guide de l'Alsace — un RAG qui se mesure

> **État : projet en cours.** Les résultats sont remplis au fil des mesures. Une case « à mesurer » n'a pas encore de chiffre.

Un assistant répond aux questions sur l'Alsace à partir de documents publics, cite ses sources, et refuse quand la réponse n'est pas dans ses documents. Chaque étage est mesuré.

## Ce que le projet montre

- Corpus public, avec la source et la licence de chaque document
- Découpage, embeddings, base vectorielle locale
- Recherche par le sens, par mots-clés, et hybride, comparées
- Réponses avec sources, refus des questions hors corpus
- Jeu de 80 questions, dont 20 pièges
- Tableau avant / après de chaque amélioration

## Résultats mesurés

| Mesure | Score |
|---|---|
| Bon document dans les 5 premiers | à mesurer |
| Réponses justes | à mesurer |
| Pièges refusés | à mesurer |
| Accord entre le juge et la notation à la main | à mesurer |
| Durée par question | à mesurer |

- Date de la mesure : à compléter
- Modèle : à compléter
- Nombre de cas : à compléter
- Commande pour refaire la mesure : à compléter

## Installation

Il faut Python 3.12, [uv](https://docs.astral.sh/uv/) et [Ollama](https://ollama.com/).

```
uv sync
copy .env.example .env
```

Puis écrire dans `.env` le nom du modèle utilisé, téléchargé avec `ollama pull`.

## Utilisation

À compléter : la commande principale et un exemple d'entrée et de sortie.

## Comment ça marche

À compléter : un schéma (Mermaid) et cinq lignes d'explication.

## Limites connues

À compléter : ce qui échoue encore, avec un ou deux exemples.

## Coût

0 €. Le projet tourne en local avec des modèles ouverts. Aucun service payant n'est nécessaire.

## Crédits

- **Code, données et mesures** : Charles Burgy.
- **Cours suivi ([COURS.md](COURS.md)) et structure de départ** : module produit par Claude Code (Anthropic), modèle Claude Fable 5.2, créé le 25/11/2026.
- **Sources officielles** : listées à la fin de [COURS.md](COURS.md).

## Licence

[MIT](LICENSE). Projet d'apprentissage, sans but commercial.
