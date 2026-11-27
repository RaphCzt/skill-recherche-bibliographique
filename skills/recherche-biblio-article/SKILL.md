---
name: "recherche-biblio-article"
description: "Recherche bibliographique (alphaXiv + vérification DBLP), évaluation de la pertinence d'une problématique et rédaction d'un article scientifique sourcé en LaTeX/BibTeX."
---

# Recherche bibliographique & rédaction d'article scientifique

Ce skill applique la méthode du cours « La Recherche Bibliographique en IA » (cycle Cadrage → Découverte → Filtrage → Synthèse → Rédaction → Pérennité) avec les outils de la session : **alphaXiv** pour la découverte et la lecture, **DBLP** pour la certification, **LaTeX/BibTeX** pour la sortie.

Deux idées gouvernent tout le reste :

1. **Une revue n'est pas un inventaire** : c'est une analyse critique comparative qui sert à *positionner* une contribution. L'article produit doit donc dire ce qui est établi, ce qui est contesté, et ce qui reste ouvert.
2. **« Faire confiance mais vérifier »** : sur arXiv tout le monde peut déposer. Aucune référence n'entre dans la bibliographie sans (a) existence vérifiée, (b) statut de publication connu, (c) passage du PDF qui soutient effectivement l'affirmation citée. Une citation inventée ou hors contexte ruine le travail entier — c'est la faute la plus grave possible ici.

## Entrées acceptées

- **Obligatoire** : une problématique (question de recherche).
- **Optionnel** : des articles déjà sélectionnés (IDs arXiv, URLs, titres, PDFs joints, dossier alphaXiv) et/ou un résumé de ce que l'utilisateur veut obtenir (angle, public, longueur, venue cible, langue).

Les articles fournis par l'utilisateur sont des *graines* (seed papers) : ils passent par le même filtrage que les autres (on le signale poliment si l'un d'eux échoue), et servent de point de départ au snowballing. Le résumé fourni oriente le plan et le ton, mais ne dispense pas de l'évaluation de pertinence.

Si seule la problématique est donnée, tout se fait à partir d'elle — ne pas bloquer pour demander des articles.

## Fichiers de travail

Les deux scripts d'aide sont fournis en **annexe A** : au début du travail, les écrire tels quels dans `biblio-<slug>/scripts/` (avec Write), ainsi que le gabarit de l'annexe B dans `biblio-<slug>/article/main.tex`.

Travailler dans un dossier `biblio-<slug-du-sujet>/` :

```
corpus.json          # une entrée par article candidat (schéma en section R1)
pertinence.md        # rapport d'évaluation de la problématique (Phase 4)
article/main.tex     # l'article
article/refs.bib     # généré par scripts/build_bib.py, jamais écrit à la main
```

`corpus.json` est la source de vérité : il est rempli au fil des phases et c'est lui qui alimente la bibliographie. Cela évite qu'une référence non vérifiée se glisse dans l'article.

Créer une liste de tâches avec les phases ci-dessous ; la dernière tâche est toujours l'audit final (Phase 7).

---

## Phase 1 — Cadrage

Avant toute recherche, reformuler la problématique en :
- **question précise** (objet, contexte, contrainte, critère de succès) ;
- **concepts clés** et leurs **synonymes / variantes de vocabulaire** — la recherche lexicale rate les articles qui utilisent d'autres mots (« hallucination » vs « unfaithful generation » vs « confabulation ») ;
- **sous-questions** (2 à 5) qui structureront la découverte et plus tard le plan ;
- **périmètre** : domaine (cs.CV, cs.LG, cs.CL…), fenêtre temporelle utile.

Si la problématique est trop vague pour être évaluée (« l'IA en médecine »), proposer 2-3 reformulations ciblées et demander laquelle retenir. Si l'utilisateur n'est pas là pour répondre, choisir la plus prometteuse et l'annoncer en tête du rapport.

## Phase 2 — Découverte (alphaXiv)

Charger les outils alphaXiv avec ToolSearch si nécessaire (`select:mcp__alphaXiv__discover_papers,mcp__alphaXiv__answer_pdf_queries,mcp__alphaXiv__get_paper_content,mcp__alphaXiv__read_files_from_github_repository`).

1. **Balayage large** — `discover_papers` est limité à 2 appels par message : couvrir toutes les facettes dans un appel large plutôt que plusieurs étroits. Faire typiquement :
   - un appel `prioritize: "historical"` pour les travaux fondateurs ;
   - un appel `prioritize: "recency"` pour l'état actuel (indispensable à l'évaluation de pertinence).
   Les `keywords` doivent être des termes réellement employés (par l'utilisateur ou vus dans des résultats), en anglais. Répéter sur les messages suivants avec les synonymes de la Phase 1 et les termes découverts.
2. **Snowballing** à partir des 2-4 articles les plus centraux (graines) :
   - *backward* : interroger le PDF (`answer_pdf_queries`) sur ses références clés / son related work pour remonter aux fondations ;
   - *forward* : chercher les travaux récents qui étendent ou contestent la graine (`discover_papers` avec le nom de la méthode en mot-clé, `prioritize: "recency"`).
3. Si l'utilisateur a un dossier ou des notes alphaXiv sur le sujet, les lire (`get_paper_notes`, `list_library`).

Viser 25-60 candidats bruts, puis consigner chaque candidat dans `corpus.json` (statut `candidat`). Ne pas tout lire : l'abstract/rapport suffit pour un premier tri.

## Phase 3 — Filtrage qualité

Pour chaque candidat retenu après lecture de l'abstract, appliquer la procédure détaillée de la **section R1** ci-dessous. En résumé :

- **Existence & statut sur DBLP.** Attention, piège important : DBLP indexe aussi les preprints arXiv sous `CoRR abs/XXXX.XXXXX` (type « Informal or Other Publication »). Être sur DBLP ne suffit donc pas : il faut regarder *quelle* entrée existe. Une entrée `conf/…` ou `journals/…` (hors `corr`) = version relue par les pairs.
- **Grille preprint du cours** pour tout ce qui n'a pas de version publiée : affiliation des auteurs, reproductibilité (code, poids, scripts — vérifiable avec `read_files_from_github_repository`), commentaire de version « Accepted to … », cohérence des résultats.
- **Statut final** de chaque article : `publie` (version relue trouvée sur DBLP), `preprint-dblp` (seulement CoRR, mais grille satisfaisante), `preprint-faible` (non indexé ou grille insuffisante → exclu des citations à l'appui d'une affirmation, mentionnable seulement comme « travaux émergents » si vraiment utile), `rejete`.

Priorité de citation : `publie` > `preprint-dblp` récent et solide. Dans l'article, un preprint est toujours présenté comme tel.

## Phase 4 — Évaluation de la pertinence de la problématique

C'est une étape décisive, pas une formalité : une recherche scientifique doit porter sur une question où il y a de la recherche en cours et un manque réel, pas sur un cas déjà résolu. Suivre la **section R2** et produire `pertinence.md` avec un verdict parmi :

- **OUVERTE** — activité récente + manque identifié et documenté → on rédige.
- **PARTIELLEMENT TRAITÉE** — le cœur est couvert, mais un angle précis reste ouvert → proposer la reformulation recentrée sur cet angle, puis rédiger sur celle-ci.
- **DÉJÀ RÉSOLUE / SATURÉE** — un ou plusieurs travaux répondent directement à la question → ne pas rédiger un article qui prétendrait à une contribution ; présenter les travaux qui la résolvent et 2-3 axes réellement ouverts adjacents.
- **INACTIVE / TROP PRÉCOCE** — pas d'activité ou pas assez de matière → le dire, proposer un recadrage.
- **TROP VAGUE** — renvoyer en Phase 1.

Chaque verdict s'appuie sur des preuves citées (articles vérifiés, sections « limitations / future work », dates). Montrer le verdict à l'utilisateur **avant** de rédiger (via SendUserMessage) quand il est autre que OUVERTE, et lui demander quel axe retenir. S'il n'est pas disponible, prendre l'axe le mieux étayé, l'écrire en tête de `pertinence.md` et continuer.

## Phase 5 — Synthèse

Lire en profondeur les 10-25 articles finalement retenus (`answer_pdf_queries` en regroupant toutes les questions d'un article dans un seul appel ; `get_paper_content` pour le rapport structuré). Pour chacun, extraire dans `corpus.json` :
- contribution, méthode, données/benchmarks, résultats chiffrés clés, limites déclarées ;
- **les affirmations qu'il soutiendra dans l'article**, avec le numéro de page et un court extrait — c'est le « Test 2 » du cours (le papier prouve-t-il bien le claim ?).

Puis construire une **matrice comparative** (articles × critères : approche, données, métrique, résultat, limite) et identifier convergences, contradictions et trous. Cette matrice devient un tableau de l'article.

## Phase 6 — Rédaction

Suivre la **section R3** et partir du gabarit de l'annexe B.

Points essentiels :
- **Type d'article honnête.** Sans résultats expérimentaux fournis par l'utilisateur, l'article est un *état de l'art critique + positionnement + méthodologie proposée* (ou position paper). Ne jamais inventer de résultats, de chiffres ou d'expériences : la section expérimentale devient un **protocole d'évaluation proposé**. Si l'utilisateur fournit des résultats, les intégrer en les attribuant clairement.
- Chaque affirmation factuelle sur un travail antérieur porte une `\cite{}` vers une entrée de `corpus.json` dont le statut est `publie` ou `preprint-dblp` et dont l'affirmation figure dans la liste des claims extraits.
- Équilibre temporel : papier(s) séminal(aux) **et** avancées récentes.
- Langue : celle de la demande, sauf venue cible anglophone ou demande contraire.
- Déclarer l'usage d'outils d'IA (section remerciements / méthodologie), comme l'exigent NeurIPS, ICML, ICLR, ACL.

Générer la bibliographie : `python3 biblio-<slug>/scripts/build_bib.py biblio-<slug>/corpus.json biblio-<slug>/article/refs.bib` (n'exporte que les entrées citables ; produit aussi le tableau récapitulatif du corpus).

## Phase 7 — Audit & pérennité

1. `python3 biblio-<slug>/scripts/check_citations.py biblio-<slug>/article/main.tex biblio-<slug>/corpus.json` — signale toute clé citée absente, non vérifiée, ou sans claim documenté. Corriger jusqu'à zéro erreur.
2. Relire chaque phrase citante contre le claim extrait : si le papier ne dit pas exactement cela, corriger la phrase (pas la citation).
3. Compiler (`latexmk -pdf main.tex` si disponible) et ouvrir le PDF pour vérifier le rendu.
4. Passer la checklist du cours (section R4).

## Livrables

Envoyer à l'utilisateur (SendUserFile) : `main.pdf` (si compilé), `main.tex`, `refs.bib`, `pertinence.md`, et un tableau du corpus (dans le message final ou en fichier). Le message final, court, donne : le verdict de pertinence en une phrase, l'axe traité, le nombre de références (publiées / preprints), et les points qui demandent une relecture humaine (ex. affirmations reposant sur un seul preprint).

Rappeler en une ligne que l'auteur reste seul garant scientifique : il doit ouvrir les PDFs des références clés avant soumission.

---

# Sections de référence

## R1 — Vérification DBLP & filtrage qualité

### Pourquoi c'est subtil
DBLP indexe **deux** types d'entrées pour un même travail :
- `journals/corr/abs-2106-09685` → « CoRR abs/2106.09685 », type *Informal or Other Publication* = **simple preprint arXiv**, aucune relecture.
- `conf/iclr/HuSWALWWC22` → « ICLR 2022 », type *Conference and Workshop Papers* = **version relue**.

Donc « présent sur DBLP » prouve l'existence et désambiguïse les auteurs, mais **seule une entrée hors `corr`** prouve la relecture par les pairs. Chercher systématiquement la version publiée, qui a parfois un titre légèrement différent ou un auteur en plus.

### Procédure (par ordre de préférence)

Le shell n'a en général pas accès au réseau et les pages dblp.org sont souvent protégées contre les robots pour WebFetch. Procéder ainsi :

1. **WebSearch restreint à DBLP** : `WebSearch(query="<titre exact> dblp", allowed_domains=["dblp.org"])`. Les extraits renvoyés contiennent les lignes DBLP du type `Auteurs: Titre. ICLR 2022` ou `CoRR abs/… (2021)` et l'URL persistante `https://dblp.org/rec/<clé>`. C'est en général suffisant. Les pages auteur (`dblp.org/pid/…`) listent souvent les deux versions côte à côte : c'est le moyen le plus rapide de voir si une version publiée existe.
   - Si rien ne remonte : réessayer avec le premier auteur + mots-clés du titre, puis sans `allowed_domains` avec « dblp » dans la requête.
2. **Navigateur** (si disponible dans la session) : ouvrir `https://dblp.org/search/publ/api?q=<titre>&format=json` ou la page `https://dblp.org/rec/<clé>.bib` pour récupérer le BibTeX exact. Charger d'abord le skill du navigateur concerné.
3. **Indices secondaires** (ne remplacent pas DBLP, mais le complètent) : commentaire de version arXiv « Accepted to … » (visible via `get_paper_content` / `answer_pdf_queries`), page OpenReview, DOI de l'éditeur.

Ne jamais deviner une clé DBLP : si elle n'a pas été vue dans un résultat, laisser `dblp_key` vide et construire la clé BibTeX localement.

Garder une trace : l'URL DBLP vue et la ligne DBLP telle qu'affichée vont dans `corpus.json` (`dblp_url`, `dblp_evidence`).

### Hors informatique
DBLP ne couvre que l'informatique. Pour un article d'un autre domaine (médecine, physique, géosciences…), l'absence sur DBLP n'est pas un signal négatif : vérifier via le DOI de l'éditeur / la revue (WebSearch), et le noter `venue_type: "journal"` avec `verification: "doi"`.

### Classement des venues
- **Tier 1** : conférences A*/A (NeurIPS, ICML, ICLR, CVPR, ICCV, ECCV, ACL, EMNLP, NAACL, KDD, AAAI, IJCAI, SIGIR, WWW…) et revues majeures (JMLR, TPAMI, IJCV, TMLR, Nature MI, AIJ…).
- **Tier 2** : autres conférences/revues indexées DBLP (ECAI, ICDCS, WACV, BMVC, ICASSP, revues IEEE/Elsevier/Springer du domaine), *Findings* of ACL/EMNLP.
- **Tier 3** : workshops.
- **Preprint** : CoRR seulement.

Pour le classement CORE d'une conférence moins connue, une recherche « <acronyme> CORE ranking » suffit.

### Grille preprint (pour tout ce qui reste en CoRR)
Noter chaque critère 0/1 dans `quality` :
- `affiliation` : labo académique ou R&D connue ;
- `code` : dépôt public avec code + scripts d'éval (vérifier avec `read_files_from_github_repository` qu'il n'est pas vide) ;
- `accepted_note` : mention d'acceptation dans les commentaires ;
- `coherence` : méthode décrite clairement, résultats comparés à des baselines à jour, pas d'affirmations démesurées ;
- `traction` : cité / repris par des travaux publiés (vu pendant le snowballing).

Seuil : ≥ 3/5 → `preprint-dblp` (si présent sur DBLP CoRR) citable comme preprint ; sinon `preprint-faible`. Un preprint de moins de ~3 mois non encore indexé peut être gardé en `preprint-faible` et cité uniquement comme « travaux très récents, non relus ».

### Statuts dans corpus.json
| statut | sens | citable à l'appui d'un claim ? |
|---|---|---|
| `candidat` | trouvé, pas encore filtré | non |
| `publie` | version relue trouvée (DBLP hors corr, ou DOI éditeur) | oui |
| `preprint-dblp` | CoRR sur DBLP + grille ≥ 3/5 | oui, présenté comme preprint |
| `preprint-faible` | non indexé ou grille < 3 | seulement comme « travaux émergents » |
| `rejete` | inexistant, hors sujet, ou douteux | non |

### Schéma de corpus.json
```json
{
  "problematique": "…",
  "papers": [
    {
      "key": "hu2022lora",
      "title": "LoRA: Low-Rank Adaptation of Large Language Models",
      "authors": ["Edward J. Hu", "Yelong Shen", "…"],
      "year": 2022,
      "arxiv_id": "2106.09685",
      "status": "publie",
      "venue": "ICLR 2022",
      "venue_full": "The Tenth International Conference on Learning Representations, ICLR 2022",
      "venue_type": "conference",
      "tier": 1,
      "dblp_key": "conf/iclr/HuSWALWWC22",
      "dblp_url": "https://dblp.org/rec/conf/iclr/HuSWALWWC22",
      "dblp_evidence": "Hu, …, Chen: LoRA: Low-Rank Adaptation of Large Language Models. ICLR 2022",
      "verification": "dblp",
      "doi": "",
      "url": "https://openreview.net/forum?id=nZeVKeeFYf9",
      "pages": "",
      "quality": {"affiliation": 1, "code": 1, "accepted_note": 1, "coherence": 1, "traction": 1},
      "role": "seminal",
      "summary": "Contribution / méthode / résultats / limites en 2-4 phrases.",
      "claims": [
        {"claim": "LoRA réduit le nombre de paramètres entraînables jusqu'à 10 000× par rapport au fine-tuning complet de GPT-3 175B", "page": 1, "evidence": "court extrait du PDF"}
      ]
    }
  ]
}
```
Champs obligatoires pour qu'une entrée soit exportée : `key`, `title`, `authors`, `year`, `status` ∈ {publie, preprint-dblp, preprint-faible}. `venue_type` ∈ {conference, journal, workshop, preprint}. `role` ∈ {seminal, baseline, extension, concurrent, survey, benchmark, other}.

## R2 — Évaluation de la pertinence d'une problématique

But : établir, preuves à l'appui, (1) qu'il y a de la recherche **en cours** sur le sujet, et (2) qu'il existe un **axe réel non traité** — pas un cas déjà résolu. Une évaluation honnête qui conclut « déjà traité » est un résultat utile : elle évite des semaines de travail inutile.

### Les six critères

Noter chaque critère (faible / moyen / fort) avec ses preuves (références du corpus, dates, extraits).

1. **Activité récente** — nombre de travaux pertinents sur les 12-24 derniers mois (appel `discover_papers` avec `prioritize: "recency"`), dont combien publiés en venue relue. Aucune activité depuis longtemps → sujet abandonné ou mûr.
2. **Réponse directe existante** — existe-t-il un travail (publié) qui répond *exactement* à la question, avec le même objet, le même contexte et des résultats solides ? Chercher activement à **réfuter** l'originalité : formuler la question comme un titre d'article plausible et le chercher, chercher avec les synonymes, chercher les surveys récents du domaine. C'est le critère le plus important ; ne pas s'arrêter à la première recherche vide.
3. **Manques déclarés** — limitations et « future work » explicitement écrits par les auteurs des travaux récents (extraits via `answer_pdf_queries` : « What limitations and future work do the authors state? »). Un manque cité par plusieurs équipes est un signal fort.
4. **Contradictions / résultats non consolidés** — résultats divergents entre articles, benchmarks non comparables, absence d'évaluation dans le contexte visé.
5. **Saturation des benchmarks** — si les méthodes atteignent le plafond des benchmarks de référence, la question « faire mieux sur X » n'a plus d'intérêt ; la question « évaluer dans un contexte plus réaliste » peut en avoir.
6. **Faisabilité / matière** — assez de travaux vérifiés pour construire un état de l'art (≥ 8-10 références citables) et des données ou benchmarks accessibles pour la méthodologie proposée.

### Règle de verdict
- **OUVERTE** : activité ≥ moyen, aucune réponse directe, ≥ 1 manque documenté par des sources vérifiées.
- **PARTIELLEMENT TRAITÉE** : la question générale a des réponses, mais un sous-cas (contexte, contrainte, population, modalité, échelle) n'est pas couvert → recentrer.
- **DÉJÀ RÉSOLUE / SATURÉE** : réponse directe publiée et solide, ou benchmarks saturés sans manque identifié.
- **INACTIVE / TROP PRÉCOCE** : activité faible et matière insuffisante.
- **TROP VAGUE** : impossible de juger sans recadrage.

En cas de doute entre deux verdicts, retenir le plus prudent et l'expliquer.

### Formuler l'axe
L'axe retenu doit être une phrase de la forme :
> « Les travaux existants [ce qui est établi, réf.] mais [manque précis, réf.]. Nous étudions [question recentrée] dans [contexte], évalué par [critère]. »

Proposer 1 axe principal et 1-2 alternatives.

### Modèle de pertinence.md

```markdown
## Évaluation de la problématique

**Problématique initiale** : …
**Reformulation cadrée** : …
**Verdict** : OUVERTE | PARTIELLEMENT TRAITÉE | DÉJÀ RÉSOLUE | INACTIVE | TROP VAGUE
**Axe retenu** : …  (si choisi sans l'utilisateur, l'indiquer ici)

### Synthèse en 3 phrases
…

### Critères
| Critère | Niveau | Preuves |
|---|---|---|
| Activité récente | fort | 14 travaux 2025-2026 dont 6 publiés (ICCV 2025, …) |
| Réponse directe existante | non trouvée | requêtes testées : … ; plus proche : [clé], qui diffère par … |
| Manques déclarés | fort | [clé1] p.8, [clé2] §6 : … |
| Contradictions | moyen | … |
| Saturation | faible | … |
| Faisabilité | fort | 18 réf. citables, datasets X, Y publics |

### Travaux les plus proches (et en quoi ils diffèrent)
- [clé] (venue, année) — …

### Axes alternatifs
1. …
2. …

### Requêtes effectuées
(liste des mots-clés / angles testés, pour la reproductibilité)
```

## R3 — Structure de l'article

### Choisir le type d'article
| Situation | Type | Section « expériences » |
|---|---|---|
| Problématique seule, pas de résultats | État de l'art critique + positionnement + méthodologie proposée | **Protocole d'évaluation proposé** (datasets, métriques, baselines, hypothèses testables) — aucun résultat chiffré inventé |
| L'utilisateur veut une revue | Survey / revue systématique | Méthodologie de la revue (requêtes, critères d'inclusion, comptes par étape) |
| L'utilisateur fournit des résultats | Article de recherche complet | Résultats fournis, attribués, jamais extrapolés |

Le titre et l'abstract doivent refléter honnêtement le type (« Vers… », « Un état de l'art et un protocole pour… », « Survey of… »).

### Plan type (état de l'art + positionnement)
1. **Titre**, auteurs (laisser les auteurs à l'utilisateur : placeholder), **résumé** (150-250 mots : contexte, manque, contribution, plan) et **mots-clés**.
2. **Introduction** — contexte et enjeu ; problématique ; manque identifié (repris de `pertinence.md`, avec citations) ; contributions sous forme de liste ; plan.
3. **Contexte / notions fondamentales** — définitions et travaux séminaux nécessaires à la compréhension.
4. **État de l'art** — organisé **par approche ou par sous-question**, jamais article par article. Pour chaque famille : principe, représentants, forces, limites, comparaison. Inclure la **matrice comparative** (tableau `table*` ou `tabularx`).
5. **Discussion critique & verrous ouverts** — convergences, contradictions, manques ; c'est ici que l'axe est justifié.
6. **Approche / méthodologie proposée** — comment l'axe pourrait être traité ; hypothèses ; positionnement par rapport aux plus proches travaux.
7. **Protocole d'évaluation proposé** — données, métriques, baselines à jour (pas obsolètes), ablations prévues, risques.
8. **Conclusion & perspectives**.
9. **Déclaration d'usage d'outils d'IA** (remerciements ou note) : préciser que la recherche documentaire et le premier jet ont été assistés par un LLM avec alphaXiv, et que les références ont été vérifiées sur DBLP.
10. **Références** (`\bibliography{refs}`).

Longueur par défaut : 6-10 pages en format article deux colonnes, ou ce que l'utilisateur demande / ce qu'impose la venue cible.

### Règles de rédaction et de citation
- Chaque affirmation sur un travail antérieur → `\cite{clé}` d'une entrée `publie` ou `preprint-dblp` de `corpus.json`, et l'affirmation doit correspondre à un `claim` extrait. Si la phrase dit plus que le claim, réduire la phrase.
- Les chiffres repris d'un article sont recopiés tels quels depuis le claim (unités, conditions, dataset), jamais arrondis de mémoire.
- Un preprint est signalé dans le texte (« dans un preprint récent, X et al. proposent… ») et dans la bibliographie (`note = {Preprint}` via `@misc`/`@article` CoRR).
- Les `preprint-faible` ne soutiennent jamais une affirmation ; ils peuvent apparaître dans une phrase du type « des travaux très récents non encore relus explorent également… ».
- Pas de citation « de remplissage » : chaque référence apporte quelque chose.
- Équilibre temporel : au moins un travail séminal et une majorité de travaux de moins de 3 ans pour un sujet actif.
- Style académique, phrases affirmatives mais nuancées ; éviter les superlatifs non sourcés (« révolutionnaire », « state of the art inégalé »).
- Langue : celle de la demande. En français, garder les termes techniques anglais usuels en italique la première fois (*fine-tuning*).

### LaTeX
Partir du gabarit de l'annexe B (classe `article`, deux colonnes, `natbib`, `babel` selon la langue). Si l'utilisateur vise une venue précise (IEEE, ACM, NeurIPS…), adapter la classe si elle est installée (`kpsewhich IEEEtran.cls`), sinon garder le gabarit et le signaler.
Compiler avec `latexmk -pdf -interaction=nonstopmode main.tex` ; corriger les warnings de citation non définie (ils signalent une clé absente de refs.bib).

## R4 — Rappels du cours : paysage éditorial et checklist
### Paysage éditorial en IA
| Type | Exemples | Relecture | Délai / rôle |
|---|---|---|---|
| Conférences A*/A | NeurIPS, ICML, ICLR, CVPR, ACL, KDD | double aveugle, 18-25 % d'acceptation | 4-8 mois, cœur de l'évaluation |
| Revues | JMLR, IEEE TPAMI, Nature MI | révisions multiples | 1-2 ans, synthèses fondatrices |
| Preprints arXiv | cs.LG, cs.AI, cs.CL, cs.CV, stat.ML | modération minimale, **pas de peer-review** | 24-48 h, vitesse de l'art |

En IA, les conférences A* valent plus que la plupart des journaux, et arXiv dicte le rythme réel.

### Checklist finale (avant d'intégrer une référence)
- [ ] Existence vérifiée (DBLP ou Scholar).
- [ ] Statut de publication identifié (preprint arXiv vs actes officiels).
- [ ] Fidélité de l'affirmation : le papier prouve ce qui est écrit.
- [ ] Équilibre temporel : séminal ET récent.
- [ ] BibTeX certifié : aucun champ manquant, DOI renseigné si existant.
- [ ] Transparence : outils d'IA utilisés mentionnés.

---

# Annexe A — Scripts

### build_bib.py
```python
#!/usr/bin/env python3
"""Génère refs.bib (et un tableau du corpus) à partir de corpus.json.

Usage : python3 build_bib.py corpus.json refs.bib [--table corpus.md]

Seules les entrées de statut publie / preprint-dblp / preprint-faible sont exportées.
Les entrées incomplètes sont signalées et ignorées plutôt qu'exportées à moitié.
"""
import argparse
import json
import re
import sys

EXPORTABLE = {"publie", "preprint-dblp", "preprint-faible"}
REQUIRED = ["key", "title", "authors", "year", "status"]


def esc(text):
    """Échappe les caractères LaTeX problématiques et protège les majuscules du titre."""
    text = str(text)
    for a, b in [("&", r"\&"), ("%", r"\%"), ("#", r"\#"), ("_", r"\_")]:
        text = re.sub(r"(?<!\\)" + re.escape(a), lambda m: b, text)
    return text


def protect_title(title):
    # Protège acronymes et mots avec majuscule interne (LoRA, YOLO, BERT) contre la mise en minuscules.
    return re.sub(r"\b([A-Za-z]*[A-Z][A-Za-z0-9]*[A-Z0-9][A-Za-z0-9]*)\b", r"{\1}", esc(title))


def entry(p):
    status = p["status"]
    vtype = p.get("venue_type", "preprint" if status != "publie" else "")
    fields = {
        "author": " and ".join(p["authors"]),
        "title": "{" + protect_title(p["title"]) + "}",
        "year": str(p["year"]),
    }
    arxiv = p.get("arxiv_id", "")
    if status == "publie" and vtype in ("conference", "workshop"):
        etype = "inproceedings"
        fields["booktitle"] = esc(p.get("venue_full") or p.get("venue", ""))
    elif status == "publie" and vtype == "journal":
        etype = "article"
        fields["journal"] = esc(p.get("venue_full") or p.get("venue", ""))
        for f in ("volume", "number"):
            if p.get(f):
                fields[f] = str(p[f])
    else:
        etype = "misc"
        if arxiv:
            fields["eprint"] = arxiv
            fields["archivePrefix"] = "arXiv"
            fields["howpublished"] = f"arXiv preprint arXiv:{arxiv}"
        fields["note"] = "Preprint, non relu par les pairs"
    if p.get("pages"):
        fields["pages"] = p["pages"].replace("-", "--").replace("----", "--")
    if p.get("doi"):
        fields["doi"] = p["doi"]
    url = p.get("url") or (f"https://arxiv.org/abs/{arxiv}" if arxiv else "")
    if url:
        fields["url"] = url
    if p.get("dblp_url"):
        fields["biburl"] = p["dblp_url"]
    body = ",\n".join(f"  {k:<13}= {{{v}}}" if not v.startswith("{") else f"  {k:<13}= {v}"
                      for k, v in fields.items())
    return f"@{etype}{{{p['key']},\n{body}\n}}\n"


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("corpus")
    ap.add_argument("out_bib")
    ap.add_argument("--table", help="écrit aussi un tableau Markdown du corpus")
    a = ap.parse_args()

    data = json.load(open(a.corpus, encoding="utf-8"))
    papers = data["papers"] if isinstance(data, dict) else data
    out, problems, keys = [], [], set()
    for p in papers:
        missing = [f for f in REQUIRED if not p.get(f)]
        if p.get("status") not in EXPORTABLE:
            continue
        if missing:
            problems.append(f"{p.get('key', p.get('title', '?'))}: champs manquants {missing}")
            continue
        if p["key"] in keys:
            problems.append(f"{p['key']}: clé dupliquée")
            continue
        if p["status"] == "publie" and not p.get("venue"):
            problems.append(f"{p['key']}: statut publie sans venue")
        keys.add(p["key"])
        out.append(entry(p))

    with open(a.out_bib, "w", encoding="utf-8") as f:
        f.write("% Généré par build_bib.py depuis corpus.json — ne pas éditer à la main\n\n")
        f.write("\n".join(out))

    counts = {}
    for p in papers:
        counts[p.get("status", "?")] = counts.get(p.get("status", "?"), 0) + 1
    print(f"{len(out)} entrées exportées vers {a.out_bib}")
    print("Statuts du corpus :", ", ".join(f"{k}={v}" for k, v in sorted(counts.items())))

    if a.table:
        rows = ["| Clé | Titre | Venue | Année | Statut | Rôle | Vérif. |", "|---|---|---|---|---|---|---|"]
        order = {"publie": 0, "preprint-dblp": 1, "preprint-faible": 2, "rejete": 3, "candidat": 4}
        for p in sorted(papers, key=lambda p: (order.get(p.get("status"), 9), -int(p.get("year") or 0))):
            rows.append("| {} | {} | {} | {} | {} | {} | {} |".format(
                p.get("key", ""), p.get("title", "").replace("|", "/"), p.get("venue", "CoRR" if p.get("arxiv_id") else ""),
                p.get("year", ""), p.get("status", ""), p.get("role", ""), p.get("verification", "")))
        open(a.table, "w", encoding="utf-8").write("\n".join(rows) + "\n")
        print(f"Tableau écrit dans {a.table}")

    if problems:
        print("\nPROBLÈMES :", *problems, sep="\n  - ")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

### check_citations.py
```python
#!/usr/bin/env python3
"""Audit des citations d'un article LaTeX contre corpus.json.

Usage : python3 check_citations.py main.tex corpus.json

Erreurs (code de sortie 1) :
  - clé citée absente du corpus ;
  - clé citée de statut non citable (candidat, rejete) ;
  - clé citée sans aucun claim documenté (Test 2 du cours non fait).
Avertissements :
  - preprint-faible cité (doit être présenté comme travail émergent, jamais à l'appui d'un claim) ;
  - entrées citables du corpus jamais citées ;
  - aucun travail séminal ou aucun travail récent (équilibre temporel) ;
  - déclaration d'usage d'outils d'IA absente.
"""
import datetime
import json
import re
import sys

CITE_RE = re.compile(r"\\(?:cite|citep|citet|citealp|citeauthor|citeyear)\*?(?:\[[^\]]*\])*\{([^}]*)\}")


def main():
    if len(sys.argv) != 3:
        print(__doc__)
        sys.exit(2)
    tex = open(sys.argv[1], encoding="utf-8").read()
    tex_nc = re.sub(r"(?<!\\)%.*", "", tex)  # ignore les commentaires
    data = json.load(open(sys.argv[2], encoding="utf-8"))
    papers = {p["key"]: p for p in (data["papers"] if isinstance(data, dict) else data) if p.get("key")}

    cited = []
    for m in CITE_RE.finditer(tex_nc):
        cited += [k.strip() for k in m.group(1).split(",") if k.strip()]
    uniq = sorted(set(cited))

    errors, warns = [], []
    for k in uniq:
        p = papers.get(k)
        if not p:
            errors.append(f"{k}: cité mais absent de corpus.json (référence non vérifiée ou inventée ?)")
            continue
        st = p.get("status")
        if st in ("candidat", "rejete", None):
            errors.append(f"{k}: statut '{st}' non citable")
        if not p.get("claims"):
            errors.append(f"{k}: aucun claim documenté (page + extrait) — vérifier ce que le papier prouve")
        if st == "preprint-faible":
            warns.append(f"{k}: preprint faible cité — à présenter comme travail émergent non relu")

    citable = [k for k, p in papers.items() if p.get("status") in ("publie", "preprint-dblp")]
    unused = [k for k in citable if k not in uniq]
    if unused:
        warns.append("entrées citables jamais citées : " + ", ".join(unused))

    years = [int(papers[k]["year"]) for k in uniq if k in papers and str(papers[k].get("year", "")).isdigit()]
    this_year = datetime.date.today().year
    if years:
        if not any(papers[k].get("role") == "seminal" for k in uniq if k in papers):
            warns.append("aucun travail marqué 'seminal' n'est cité")
        if not any(y >= this_year - 2 for y in years):
            warns.append("aucun travail de moins de 3 ans n'est cité")
    if not re.search(r"(outils? d'IA|AI (tools?|assistance)|LLM|modèle de langage)", tex_nc, re.I):
        warns.append("pas de déclaration d'usage d'outils d'IA détectée")

    n_pub = sum(1 for k in uniq if papers.get(k, {}).get("status") == "publie")
    n_pre = sum(1 for k in uniq if papers.get(k, {}).get("status") in ("preprint-dblp", "preprint-faible"))
    print(f"{len(uniq)} références citées ({n_pub} publiées, {n_pre} preprints), {len(cited)} occurrences.")
    for w in warns:
        print("AVERTISSEMENT:", w)
    for e in errors:
        print("ERREUR:", e)
    if errors:
        sys.exit(1)
    print("OK : toutes les citations sont vérifiées et documentées.")


if __name__ == "__main__":
    main()
```

# Annexe B — Gabarit LaTeX (main.tex)
```latex
\documentclass[10pt,twocolumn,a4paper]{article}
% --- Langue : remplacer french par english si l'article est en anglais.
% Si babel-french n'est pas installé, on retombe sur l'anglais (césure seulement).
\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\IfFileExists{french.ldf}{\usepackage[french]{babel}}{\IfFileExists{english.ldf}{\usepackage[english]{babel}}{}%
  % article en français sans babel-french : titres traduits à la main (supprimer pour un article anglais)
  \AtBeginDocument{\renewcommand{\abstractname}{Résumé}\renewcommand{\refname}{Références}\renewcommand{\tablename}{Tableau}}}
\usepackage{lmodern}
\usepackage[margin=2cm]{geometry}
\usepackage{amsmath,amssymb}
\usepackage{graphicx}
\usepackage{booktabs,tabularx}
\usepackage[numbers,sort&compress]{natbib}
\usepackage[hidelinks]{hyperref}
\usepackage{xcolor}

\title{%TITRE%}
\author{%AUTEURS% \\ \small %AFFILIATION%}
\date{%DATE%}

\begin{document}
\twocolumn[
  \begin{@twocolumnfalse}
    \maketitle
    \begin{abstract}
    %RESUME%
    \end{abstract}
    \noindent\textbf{Mots-clés :} %MOTS-CLES%
    \vspace{1em}
  \end{@twocolumnfalse}
]

\section{Introduction}
% Contexte, problématique, manque identifié (citations), contributions (liste), plan.

\section{Notions fondamentales}

\section{État de l'art}
% Organisé par approche / sous-question. Matrice comparative :
% \begin{table*}[t]\centering\small
% \caption{Comparaison des principales approches.}
% \begin{tabularx}{\textwidth}{@{}lXXXX@{}}\toprule
% Travail & Approche & Données & Résultat clé & Limite \\\midrule
% \bottomrule\end{tabularx}\end{table*}

\section{Discussion critique et verrous ouverts}

\section{Approche proposée}

\section{Protocole d'évaluation proposé}
% Aucun résultat inventé : données, métriques, baselines, hypothèses testables.

\section{Conclusion et perspectives}

\section*{Déclaration d'usage d'outils d'IA}
La recherche documentaire et la rédaction d'un premier jet ont été assistées par un modèle de langage (Claude) interrogeant la base alphaXiv. Chaque référence a été vérifiée sur DBLP (statut de publication) et chaque affirmation citée a été confrontée au texte source. Les auteurs restent seuls responsables du contenu.

\bibliographystyle{plainnat}
\bibliography{refs}

\end{document}
```