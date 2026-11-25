# Cours 2 — Guide de l'Alsace : un RAG qui se mesure

> **Module de cours produit par Claude Code (Anthropic), modèle Claude Fable 5.2. Créé le 25/11/2026**, à partir des sources officielles listées en fin de document. Le cours donne le chemin ; le code du projet est écrit par Charles Burgy.

| | |
|---|---|
| **Durée estimée** | 3 semaines, soit 12 à 16 séances de 1 à 2 h |
| **Niveau** | Intermédiaire. Le projet 1 est fait |
| **Coût** | 0 €. Modèle de texte et modèle d'embeddings en local |
| **Avant de commencer** | Guide 04 relu |

## Le projet en une phrase

Un assistant répond aux questions sur l'Alsace (appellations, cépages, grands crus, villages, histoire) à partir de documents publics, cite ses sources, et refuse quand la réponse n'est pas dans ses documents.

## Le sigle

RAG veut dire « retrieval-augmented generation » : on cherche d'abord les passages utiles dans des documents, puis on demande au modèle de répondre à partir de ces passages seulement. Le modèle n'a pas besoin de « savoir » : il lit.

## Pourquoi ce projet est différent d'un RAG de plus

Un RAG se monte en une heure avec un tutoriel. Presque personne ne mesure le sien. Ici, la moitié du cours porte sur la mesure : la recherche trouve-t-elle le bon passage, la réponse est-elle juste, les pièges sont-ils refusés.

## Ce que tu sauras faire à la fin

- Constituer un corpus en respectant les licences.
- Découper des textes et les transformer en vecteurs (embeddings).
- Chercher par le sens, par les mots, et en combinant les deux.
- Faire répondre un modèle avec ses sources, et le faire refuser.
- Évaluer chaque étage séparément, et prouver qu'une amélioration en est une.

## L'arborescence visée

```
02-guide-alsace/
├── guide_alsace/
│   ├── config.py
│   ├── llm.py           repris du projet 1
│   ├── corpus.py        téléchargement, nettoyage
│   ├── decoupage.py     texte -> passages
│   ├── index.py         passages -> base vectorielle
│   ├── recherche.py     question -> passages
│   ├── reponse.py       question + passages -> réponse avec sources
│   └── cli.py
├── tests/
├── data/
│   ├── sources.csv      adresse, titre, licence, date de téléchargement
│   └── brut/            les textes téléchargés (ignorés par git)
└── eval/
    ├── questions.csv
    ├── evaluer_recherche.py
    ├── evaluer_reponses.py
    └── resultats/
```

---

## Séance 0 — Mise en place

1. `uv sync`, puis copier `.env.example` en `.env`.
2. Télécharger le modèle d'embeddings indiqué dans le guide 02.
3. Copier `config.py` et `llm.py` depuis le projet 1. Noter dans le journal ce qu'il a fallu adapter.
4. Créer le dépôt git local.

---

## Module 1 — Le corpus et ses licences

**Objectif** : 30 à 60 documents publics sur l'Alsace, dont tu sais d'où ils viennent et ce que tu as le droit d'en faire.

**Notions**

- Une licence dit ce qu'on peut faire d'un texte. Wikipédia est sous CC BY-SA : réutilisation permise si l'on cite la source et si l'on partage sous la même licence. Les données de data.gouv.fr sont le plus souvent sous Licence Ouverte.
- Un site sans mention de licence n'est pas libre. On ne le prend pas.
- On publie sur GitHub la liste des sources et le script qui les télécharge, pas les textes. Le dépôt reste léger et sans souci de droits.

**À faire**

1. Lister les sources dans `data/sources.csv` : adresse, titre, licence, date. Pistes : cahiers des charges des appellations d'Alsace (INAO), articles Wikipédia sur les cépages, les grands crus et les villages, pages Wikivoyage, jeux de données de data.gouv.fr.
2. Écrire `corpus.py` : il lit `sources.csv` et enregistre chaque texte dans `data/brut/`. Pour Wikipédia, utiliser l'API officielle plutôt que de lire le HTML.
3. Respecter les serveurs : une pause entre deux téléchargements, un nom de programme clair dans l'en-tête `User-Agent`.
4. Nettoyer : retirer menus, notes de bas de page, tableaux illisibles.

**Réussi quand** : `uv run python -m guide_alsace.corpus` reconstitue tout le corpus à partir de `sources.csv`.

**Piège** : un PDF de cahier des charges donne un texte plein de coupures de ligne et d'en-têtes répétés. Regarder le texte obtenu avant d'aller plus loin.

---

## Module 2 — Découper

**Objectif** : transformer chaque document en passages de taille raisonnable.

**Notions**

- Un passage (« chunk ») est l'unité qu'on cherche et qu'on donne au modèle. Trop court, il perd son contexte. Trop long, il noie l'information et ralentit le modèle.
- Le chevauchement : deux passages voisins partagent quelques phrases, pour ne pas couper une idée en deux.
- Chaque passage garde ses papiers : document d'origine, titre de section, adresse.

**À faire**

1. Écrire `decoupage.py` avec deux méthodes : par nombre de caractères, puis par paragraphes et titres.
2. Tester avec pytest : aucun passage vide, aucun passage au-dessus de la taille maximum, tout le texte d'origine se retrouve dans les passages.
3. Lire 20 passages au hasard. Sont-ils compréhensibles seuls ?

**Réussi quand** : chaque passage affiché indique son document et sa section.

---

## Module 3 — Les embeddings, à la main d'abord

**Objectif** : comprendre ce qu'est un vecteur de sens avant d'utiliser une base qui les cache.

**Notions**

- Un embedding est une liste de nombres qui représente le sens d'un texte. Deux textes de sens proche ont des listes proches.
- La similarité cosinus mesure cette proximité : 1 pour deux vecteurs de même direction, 0 pour deux vecteurs sans rapport.
- Le modèle d'embeddings est un autre modèle que celui qui rédige. Il est petit et rapide.

**À faire**

1. Écrire 6 phrases : 3 sur le vin, 2 sur l'architecture alsacienne, 1 sur le football.
2. Obtenir leur embedding par Ollama, et regarder la taille du vecteur.
3. Calculer avec numpy la similarité cosinus de chaque paire, et afficher le tableau 6 × 6.
4. Poser une question et classer les 6 phrases par proximité.
5. Essayer la même question en allemand. Le classement tient-il ?

**Réussi quand** : tu peux expliquer le tableau à quelqu'un qui ne connaît pas le sujet.

---

## Module 4 — La base vectorielle

**Objectif** : ranger tous les passages et retrouver les 5 plus proches d'une question.

**Notions**

- Une base vectorielle stocke les vecteurs avec leur texte et leurs papiers, et sait chercher les plus proches rapidement. Chroma en est une, libre, qui écrit dans un simple dossier local.
- L'index se construit une fois, puis se réutilise. Si le découpage ou le modèle d'embeddings change, il faut le reconstruire.

**À faire**

1. `uv add chromadb`.
2. Écrire `index.py` : il lit les passages, calcule les embeddings et remplit la base. Il affiche sa progression et sa durée.
3. Écrire dans `recherche.py` une fonction `chercher(question, k=5)`.
4. Poser 10 questions et lire les passages rendus.

**Réussi quand** : la commande de recherche affiche 5 passages avec leur source et leur score.

---

## Module 5 — Le jeu d'évaluation, avant d'aller plus loin

**Objectif** : pouvoir chiffrer la qualité de la recherche.

**Pourquoi maintenant** : si la recherche ne trouve pas le bon passage, aucun modèle ne rattrapera la réponse. On mesure donc cet étage seul, avant de brancher la génération.

**À faire**

1. Écrire `eval/questions.csv`, 80 questions :
   - 60 dont la réponse est dans le corpus, avec la réponse attendue et le document attendu ;
   - 20 pièges : hors corpus (une question sur la Bourgogne), hors sujet (la météo de demain), ou fondées sur une idée fausse.
2. Varier : questions précises, questions vagues, fautes de frappe, mots différents de ceux du texte.
3. Mettre 16 questions de côté (colonne `reserve`), à ne regarder qu'à la fin.
4. Écrire `evaluer_recherche.py` : pour chaque question, le document attendu est-il dans les 5 premiers passages ? Dans le premier ?
5. Noter le score de départ dans le journal.

**Réussi quand** : une commande sort les deux scores et la liste des questions ratées.

---

## Module 6 — Chercher par les mots, puis les deux

**Objectif** : améliorer la recherche, mesure à l'appui.

**Notions**

- La recherche par le sens rate les noms propres et les sigles rares : « Kaefferkopf » ne ressemble à rien pour un modèle d'embeddings.
- La recherche par mots-clés (méthode BM25) fait l'inverse : parfaite sur le mot exact, aveugle aux synonymes.
- La recherche hybride lance les deux et fusionne les classements. Une méthode simple : la fusion par rang réciproque (chaque passage reçoit des points selon sa place dans chaque liste).

**À faire**

1. `uv add rank-bm25`, puis écrire la recherche par mots-clés. Attention à la préparation du texte : minuscules, accents, ponctuation.
2. Écrire la fusion des deux classements.
3. Mesurer les trois méthodes sur le même jeu.

| Méthode | Bon document dans les 5 premiers | Bon document en premier |
|---|---|---|
| Par le sens | | |
| Par mots-clés | | |
| Hybride | | |

**Réussi quand** : le tableau est rempli avec tes chiffres, et tu sais dire quelles questions chaque méthode rate.

---

## Module 7 — Répondre, avec sources, ou refuser

**Objectif** : une réponse appuyée sur les passages, ou un refus net.

**Notions**

- Le prompt contient la consigne, les passages numérotés, puis la question.
- La consigne dit : répondre seulement à partir des passages, citer le numéro du passage après chaque affirmation, et répondre « Je ne trouve pas cette information dans mes documents » sinon.
- La sortie est structurée, comme au projet 1 : la réponse, la liste des passages cités, un indicateur de refus.
- La fenêtre de contexte est la quantité de texte que le modèle peut lire en une fois. Un petit modèle local en a une petite : 5 passages, pas 30.

**À faire**

1. Écrire `reponse.py` avec une sortie Pydantic.
2. Contrôler par le code : chaque passage cité existe bien dans ceux qui ont été fournis.
3. Ajouter un seuil : si aucun passage n'est assez proche, refuser sans appeler le modèle. Régler ce seuil sur tes mesures, pas au hasard.
4. Écrire `cli.py` : une question en entrée ; la réponse, les sources et leur adresse en sortie.

**Réussi quand** : une question sur la Bourgogne est refusée, et une question sur le riesling reçoit une réponse avec au moins une source vérifiable.

---

## Module 8 — Évaluer les réponses

**Objectif** : trois scores pour le système complet.

**À faire**

1. Écrire `evaluer_reponses.py` :
   - **justesse** : la réponse correspond-elle à la réponse attendue ?
   - **refus** : les 20 pièges sont-ils refusés ? Et combien de vraies questions sont refusées à tort ?
   - **sources** : le document cité est-il le document attendu ?
2. Pour la justesse, utiliser un modèle juge qui répond par oui ou par non (guide 04, section 5).
3. Noter 20 réponses à la main, comparer avec le juge, et écrire le taux d'accord.
4. Mesurer la durée moyenne par question.

**Réussi quand** : le tableau des trois scores est dans le README, avec le taux d'accord du juge.

---

## Module 9 — Améliorer, une chose à la fois

**Objectif** : un tableau « avant / après » honnête.

**À faire** : choisir 3 idées, les tester l'une après l'autre sur le même jeu, garder celles qui aident.

- Changer la taille des passages et le chevauchement.
- Reclasser : récupérer 20 passages, demander à un modèle de garder les 5 meilleurs.
- Reformuler la question avant la recherche.
- Ajouter le titre du document au début de chaque passage.
- Changer de modèle d'embeddings.

À la fin seulement, lancer le système retenu sur les 16 questions mises de côté. Le score tient-il ?

**Réussi quand** : chaque ligne du tableau donne l'idée, le score avant, le score après, la durée par question, et la décision (gardée ou non).

---

## Module 10 — Finir le dépôt

1. Une petite interface, au choix : la commande suffit ; une page Streamlit est un plus.
2. Le schéma du système en Mermaid : question, recherche, passages, modèle, réponse.
3. Le README complet, avec la section « Sources et licences » qui renvoie à `sources.csv`.

## Le livrable

- [ ] Le dépôt, sans les textes du corpus, avec `sources.csv` et le script de téléchargement
- [ ] Le jeu de 80 questions
- [ ] Le tableau des trois méthodes de recherche
- [ ] Les scores du système complet et le taux d'accord du juge
- [ ] Le tableau avant / après des améliorations
- [ ] Ce qui échoue encore, avec deux exemples

## Ce que le projet 3 réutilise

`llm.py`, les sorties structurées, la méthode « mesurer chaque étage ».

## Sources officielles

- Embeddings avec Ollama : https://docs.ollama.com/capabilities/embeddings
- Sorties structurées avec Ollama : https://docs.ollama.com/capabilities/structured-outputs
- Chroma : https://docs.trychroma.com/
- rank-bm25 : https://pypi.org/project/rank-bm25/
- NumPy : https://numpy.org/doc/stable/
- Streamlit : https://docs.streamlit.io/
- API de Wikipédia (MediaWiki) : https://www.mediawiki.org/wiki/API:Action_API
- Licence CC BY-SA 4.0 : https://creativecommons.org/licenses/by-sa/4.0/deed.fr
- Licence Ouverte (Etalab) : https://www.data.gouv.fr/pages/legal/licences/etalab-2-0
- Données publiques : https://www.data.gouv.fr/
- Cahiers des charges des appellations (INAO) : https://www.inao.gouv.fr/rechercher-un-produit
