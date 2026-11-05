# Lab 03 | Concevoir deux agents UI dans Cursor

**Agentic SDLC · Atelier Design & Prototype**  
**Durée : 2 h 15 · dont 90 minutes de pratique**  
**Niveau : débutant à intermédiaire · Format : groupes de 2 ou 3**

> **Objectif**  
> Concevoir dans Cursor deux sous-agents qui transforment un fichier de spécifications Markdown en écrans HTML, puis en prototype interactif. À la fin, les participants disposent d'un prototype démontrable et d'un dossier de transmission pour les ateliers Qualité & Tests et Architecture & Code.

Dans ce lab, chaque étape précise **quoi faire**, **quoi saisir**, **quoi observer** et **comment vérifier le résultat**. Les deux agents finaux de référence sont disponibles dans le dossier `solutions/.cursor/agents/`.

## Compétences acquises

À la fin de cet atelier, vous saurez :

1. créer un sous-agent Cursor dans `.cursor/agents/` ;
2. définir sa mission, ses entrées, ses règles et ses livrables ;
3. transmettre les requirements, le design system et le code existant à un agent ;
4. utiliser `design-generator` pour créer les maquettes HTML ;
5. utiliser `proto-builder` pour créer un prototype interactif autonome ;
6. contrôler la couverture des exigences et transmettre un dossier utilisable par un agent de développement.

## Avant de commencer

### Ce qu'il vous faut

- Cursor installé, avec un modèle accessible en mode **Agent**.
- Un projet contenant la base de code de l'application existante, accessible en lecture.
- Un fichier de spécifications détaillées, par exemple `docs/requirements.md`.
- Un design system **déjà existant** au format Markdown, par exemple `docs/design-system.md`.
- Le droit de créer des fichiers dans le dossier de travail.
- Facultatif : Git pour vérifier que l'application de production n'a pas été modifiée.

**Important :** les spécifications peuvent ne pas contenir de partie tests. Le but du lab est de concevoir et simuler l'expérience, pas d'écrire les tests de production.

### Structure de départ recommandée

```text
mon-application/
├── .cursor/
│   └── agents/                  # à créer
├── docs/
│   ├── requirements.md          # fourni
│   └── design-system.md         # fourni
├── src/                         # code applicatif existant, selon le projet
└── ...
```

Les chemins du lab sont **des exemples**. Si votre équipe utilise un autre emplacement, remplacez les chemins dans les prompts. La racine de la base de code peut être `.` et n'est pas nécessairement `src/`.

> **Conseil de l'animateur**  
> Avant l'atelier, mettez à disposition des groupes les mêmes documents d'entrée et une version de travail de l'application. Évitez les données de clients et les secrets. Les participants ne doivent pas créer un nouveau design system.

## Le parcours de l'atelier

| Temps | Activité | Nature |
|---|---|---|
| 00:00–00:10 | Comprendre le résultat visé | Introduction |
| 00:10–00:20 | Observer une démonstration cible | Démonstration |
| 00:20–00:30 | Vérifier les documents et le projet | Préparation |
| 00:30–00:40 | Comprendre la structure d'un sous-agent Cursor | Explication |
| 00:40–01:00 | Construire `design-generator` | Pratique, 20 min |
| 01:00–01:15 | Générer et contrôler les maquettes | Pratique, 15 min |
| 01:15–01:30 | Construire `proto-builder` | Pratique, 15 min |
| 01:30–01:55 | Créer le prototype et le dossier de transmission | Pratique, 25 min |
| 01:55–02:10 | Tester entre groupes et contrôler le dossier | Pratique, 15 min |
| 02:10–02:15 | Faire le bilan | Conclusion |

---

## Exercice 1 · Préparer l'espace de travail

**Durée : 10 minutes**

### Étape 1. Ouvrir le bon dossier

1. Ouvrez Cursor.
2. Sélectionnez **File > Open Folder** et ouvrez la racine de l'application existante.
3. Vérifiez que les fichiers `requirements.md` et `design-system.md` sont accessibles dans ce dossier ou dans un dossier du même espace de travail.
4. Ouvrez le panneau de chat en mode **Agent**, et non uniquement le mode conversation sans outils.

Dans le terminal intégré, vérifiez la présence des entrées (adaptez les chemins si nécessaire) :

```bash
test -f docs/requirements.md && echo "Requirements : OK"
test -f docs/design-system.md && echo "Design system : OK"
```

**Résultat attendu :** les deux lignes affichent `OK`, et l'application existante est visible dans Cursor.

> **Si un fichier manque**  
> N'en demandez pas la création à l'agent. Demandez le bon chemin ou faites fournir le document manquant. Les agents doivent utiliser les documents déjà fournis.

### Étape 2. Créer le dossier des agents

Créez ce répertoire depuis l'arborescence de Cursor ou depuis le terminal :

```bash
mkdir -p .cursor/agents
```

**Résultat attendu :** `.cursor/agents/` existe dans le projet.

### Étape 3. Définir les responsabilités

En groupe, associez chaque responsabilité au bon agent :

| Responsabilité | Agent |
|---|---|
| Analyser les exigences et le design system | `design-generator` |
| Repérer les écrans, états et actions | `design-generator` |
| Générer des maquettes HTML statiques | `design-generator` |
| Rendre les parcours cliquables | `proto-builder` |
| Simuler les réponses métier sans backend | `proto-builder` |
| Préparer les documents pour l'agent de code | `proto-builder` |

**Point clé :** les deux agents ont des missions distinctes. Le second dépend des fichiers produits par le premier.

---

## Exercice 2 · Créer le premier agent : `design-generator`

**Durée : 20 minutes**

### Étape 1. Créer un sous-agent Cursor

Dans `.cursor/agents/`, créez le fichier `design-generator.md` et commencez par cette configuration :

```yaml
---
name: design-generator
description: Génère des maquettes HTML à partir des requirements, du design system Markdown fourni et de la base de code existante.
model: inherit
readonly: false
is_background: false
---
```

`name` désigne le sous-agent, `description` permet à Cursor de décider quand l'utiliser, `model: inherit` reprend le modèle de l'agent principal, et `readonly: false` l'autorise à créer les fichiers de maquettes. `is_background: false` convient ici, car le deuxième agent attend les sorties du premier. <https://cursor.com/docs/subagents>

### Étape 2. Écrire la mission du premier agent

Sous la configuration, ajoutez :

```markdown
# Mission

Tu es un spécialiste UI. Tu transformes les exigences
fonctionnelles d'une application existante en maquettes HTML.

Tu dois respecter le design system fourni et les conventions
visuelles du produit existant.

Tu ne développes pas la fonctionnalité de production.
```

### Étape 3. Compléter les instructions avec Cursor

Ouvrez un **nouveau chat Agent** dans Cursor et collez le prompt suivant :

```text
Aide-moi à compléter uniquement le fichier
.cursor/agents/design-generator.md.

Sa mission : transformer les requirements Markdown en maquettes
HTML statiques, sans modifier l'application existante.

Ajoute des sections claires :
1. Entrées : requirementsPath, designSystemPath,
   codebaseRoot, featureId et outputRoot.
2. Priorités : requirements pour le besoin,
   design system pour le visuel, code existant pour le contexte.
3. Méthode : lecture, analyse UX, liste des écrans,
   états, actions, puis génération des fichiers HTML.
4. Règles : ne pas inventer de règles métier ;
   ne jamais modifier le design system ou le code de production.
5. Sorties : galerie HTML, manifeste JSON, correspondance
   exigences-écrans et décisions UX.
6. Contrôles de fin : existence des fichiers, JSON valide,
   exigences couvertes et absence de changement en production.

Conserve les identifiants REQ-* et ACT-* pour assurer
la continuité avec le futur agent de prototypage.
Écris uniquement les instructions de cet agent.
```

Relisez le fichier créé. Ne validez pas automatiquement toutes les propositions : vérifiez qu'il n'a pas demandé de recréer le design system et qu'il écrit uniquement dans le dossier de sortie.

### Étape 4. Fixer le contrat de sortie

Le dossier de sortie doit suivre ce modèle :

```text
design-output/<featureId>/
├── screens/
│   ├── index.html
│   ├── assets/styles.css
│   ├── <screen-id>--default.html
│   └── <screen-id>--error.html       # si nécessaire
├── screens-manifest.json
├── requirements-mapping.md
├── ux-decisions.md
└── design-review.md
```

Faites ajouter explicitement à l'agent un manifeste JSON qui contient au minimum : `schemaVersion`, `featureId`, `sources`, `screens[]`, les `states[]`, les `actions[]` et les `openQuestions[]`. Chaque action doit porter un identifiant `ACT-*` et, si possible, le ou les identifiants `REQ-*` correspondants.

Voici un **exemple de structure**, à ne pas confondre avec les exigences de votre projet :

```json
{
  "schemaVersion": "1.0",
  "featureId": "exemple-fonctionnalite",
  "sources": {
    "requirementsPath": "docs/requirements.md",
    "designSystemPath": "docs/design-system.md",
    "codebaseRoot": "."
  },
  "screens": [
    {
      "id": "liste",
      "title": "Liste",
      "states": [{"id": "default", "file": "screens/liste--default.html"}],
      "requirements": ["REQ-001"],
      "actions": [
        {
          "id": "ACT-001",
          "label": "Consulter un élément",
          "trigger": "clic",
          "targetScreenId": "detail",
          "expectedBehavior": "Afficher le détail",
          "requirements": ["REQ-001"]
        }
      ]
    }
  ],
  "openQuestions": []
}
```

### Vérification du premier agent

Dans Cursor, ouvrez `design-generator.md` et assurez-vous que :

- [ ] Le fichier se trouve bien dans `.cursor/agents/`.
- [ ] Il possède un en-tête YAML valide.
- [ ] Il mentionne les trois entrées existantes.
- [ ] Il n'autorise aucune modification du code de production.
- [ ] Il définit des sorties HTML et un manifeste JSON.
- [ ] Il conserve des identifiants liés aux exigences et aux interactions.

**Corrigé de référence :** `solutions/.cursor/agents/design-generator.md`.

---

## Exercice 3 · Utiliser `design-generator` pour produire les maquettes

**Durée : 15 minutes**

### Étape 1. Appeler explicitement le sous-agent

Dans le chat principal Cursor en mode Agent, saisissez :

```text
/design-generator

Fonctionnalité : <nom-de-la-fonctionnalite>
Requirements : docs/requirements.md
Design system : docs/design-system.md
Base de code existante : .
Sortie : design-output/<featureId>/

Lis les sources. Identifie les écrans, les états et les actions.
Produis les maquettes HTML et le manifeste JSON.
Ne modifie pas les fichiers de production.
Signale les hypothèses et les exigences non couvertes.
```

> **Astuce**  
> Cursor prend en charge l'appel explicite d'un sous-agent avec `/nom-agent`. Si votre version n'affiche pas cette suggestion, utilisez : « Utilise le sous-agent design-generator pour… » et vérifiez qu'une délégation a réellement eu lieu. <https://cursor.com/docs/subagents>

### Étape 2. Ouvrir les écrans

Après exécution, ouvrez `design-output/<featureId>/screens/index.html` dans un navigateur. Si l'ouverture directe ne convient pas à votre navigateur, utilisez un serveur local simple depuis la racine du dépôt :

```bash
python3 -m http.server 8000
```

Puis ouvrez le chemin correspondant sur `http://localhost:8000/`.

### Étape 3. Contrôler le résultat

Observez :

- La cohérence visuelle avec le design system.
- La couverture du parcours principal.
- La présence des états vides, erreurs et succès lorsqu'ils sont requis.
- La présence d'un lien entre les actions et les exigences.
- Les questions ouvertes documentées dans `ux-decisions.md`.

Vérifiez le JSON :

```bash
python3 -m json.tool design-output/<featureId>/screens-manifest.json > /dev/null
```

Remplacez `<featureId>` par le nom réel du dossier, sans conserver les chevrons dans la commande.

**Résultat attendu :** une galerie de maquettes HTML et un manifeste lisible par le deuxième agent.

### Point de contrôle collectif

Le groupe présente ses écrans en une minute : quel est le parcours, où se trouvent les actions, quelles hypothèses nécessitent une validation ? **Ne passez à l'agent 2 qu'après cette revue.**

---

## Exercice 4 · Créer le deuxième agent : `proto-builder`

**Durée : 15 minutes**

### Étape 1. Créer la définition

Dans `.cursor/agents/`, créez `proto-builder.md` :

```yaml
---
name: proto-builder
description: Transforme les maquettes de design-generator en prototype interactif autonome, puis prépare un dossier pour les ateliers Qualité et Développement.
model: inherit
readonly: false
is_background: false
---
```

Ajoutez ce rôle sous l'en-tête :

```markdown
# Mission

Tu transformes des maquettes HTML en prototype UX interactif.
Tu interviens après design-generator.
Tu simules les parcours métier avec des données fictives,
sans backend réel, sans modifier le code de production.
Tu produis aussi un dossier de transmission pour les agents
chargés des tests et de la génération du code.
```

### Étape 2. Construire les instructions avec Cursor

Dans un nouveau chat Cursor, utilisez :

```text
Complète uniquement .cursor/agents/proto-builder.md.

Le sous-agent prend en entrée les requirements, le design system,
la base de code, les maquettes et screens-manifest.json
produits par design-generator.

Ajoute des consignes pour :
1. Conserver les identifiants REQ-*, ACT-* et screenId.
2. Créer un prototype autonome en HTML, CSS et JS locaux.
3. Relier les écrans et gérer les interactions réelles du prototype.
4. Simuler les données et réponses métier, sans API réelle.
5. Gérer succès, erreur, initial et autres états sourcés.
6. Ajouter une fonction de réinitialisation des scénarios.
7. Produire un guide de démo sponsor et des tâches UX.
8. Créer le dossier handoff vers les agents Qualité et Code.
9. Préparer les critères d'acceptation et un prompt
   pour le futur agent de développement.
10. Signaler toute hypothèse non validée.

N'autorise des écritures que dans design-output/<featureId>/.
N'invente pas de validations sponsor ou utilisateur.
Le prototype doit guider l'UX, pas imposer son JS à la production.
```

### Étape 3. Fixer les livrables

Demandez à l'agent d'écrire dans :

```text
design-output/<featureId>/
├── interactive/
│   ├── index.html
│   ├── assets/styles.css
│   ├── assets/app.js
│   └── README.md
├── prototype-flows.md
├── sponsor-demo.md
├── user-session-guide.md
├── ux-changes.md
└── handoff/
    ├── IMPLEMENTATION_BRIEF.md
    ├── ACCEPTANCE_CRITERIA.md
    ├── TRACEABILITY.md
    ├── handoff-manifest.json
    └── PROMPT_FOR_DEV_AGENT.md
```

Le fichier `handoff-manifest.json` doit enregistrer les chemins des requirements, du design system, des maquettes, du prototype et des documents de transmission. Son statut initial est `review_required`. Les validations sponsor/utilisateurs sont `not_yet_validated` tant qu'elles n'ont pas eu lieu.

### Vérification du deuxième agent

- [ ] Les fichiers de `design-generator` sont des entrées obligatoires.
- [ ] Les actions `ACT-*` doivent produire un comportement ou une limite documentée.
- [ ] Le prototype ne fait aucun appel à un vrai backend.
- [ ] Un scénario peut être remis à zéro.
- [ ] Le dossier contient les documents requis pour la suite du SDLC.
- [ ] Les validations humaines ne sont jamais inventées.

**Corrigé de référence :** `solutions/.cursor/agents/proto-builder.md`.

---

## Exercice 5 · Générer le prototype interactif et le handoff

**Durée : 25 minutes**

### Étape 1. Lancer le deuxième sous-agent

Dans le chat principal Cursor, saisissez :

```text
/proto-builder

Fonctionnalité : <nom-de-la-fonctionnalite>
Requirements : docs/requirements.md
Design system : docs/design-system.md
Base de code : .
Dossier de maquettes : design-output/<featureId>/

Reprends les maquettes et le screens-manifest.json.
Crée interactive/index.html avec navigation et interactions.
Utilise uniquement des données et réponses fictives.
Prévois les états requis et un bouton Réinitialiser.
Crée le script de démonstration, les tâches utilisateur
et le dossier handoff pour Qualité et Développement.
Ne modifie pas le code de production.
```

### Étape 2. Tester le parcours utilisateur

Ouvrez `design-output/<featureId>/interactive/index.html` dans le navigateur. Si nécessaire, utilisez le serveur local indiqué plus haut.

Essayez réellement :

1. de démarrer le parcours principal ;
2. de renseigner un formulaire ou d'effectuer l'action centrale ;
3. d'atteindre un état de confirmation ;
4. de produire un état d'erreur prévu ;
5. de réinitialiser le prototype ;
6. de recommencer le même parcours.

> **Note**  
> Si une action n'est pas simulable, elle doit être annoncée comme telle dans l'interface et dans le dossier de transmission. Un bouton visuellement actif mais sans effet ne constitue pas un parcours interactif valide.

### Étape 3. Lire le dossier de transmission

Ouvrez les documents suivants :

| Fichier | Utilité pour la suite |
|---|---|
| `IMPLEMENTATION_BRIEF.md` | Ce que doit faire la fonctionnalité réelle |
| `ACCEPTANCE_CRITERIA.md` | Conditions vérifiables, liées aux exigences |
| `TRACEABILITY.md` | Correspondance exigences, écrans, actions, critères |
| `handoff-manifest.json` | Index machine-lisible des livrables et du statut |
| `PROMPT_FOR_DEV_AGENT.md` | Prompt pour démarrer l'agent de développement |

Le futur agent de code **ne doit pas recopier automatiquement** le JavaScript de démonstration. Il doit analyser l'architecture réelle de l'application, puis implémenter les comportements validés.

### Étape 4. Valider la cohérence des fichiers

```bash
python3 -m json.tool design-output/<featureId>/handoff/handoff-manifest.json > /dev/null
```

Si votre projet utilise Git, vérifiez également les modifications :

```bash
git status --short
```

La présence de nouveaux fichiers sous `.cursor/agents/` et `design-output/` est normale. Les fichiers de production existants ne doivent pas avoir été modifiés par ces deux agents.

**Résultat attendu :** un prototype qui fonctionne sur le parcours principal, plus un dossier exploitable par les ateliers suivants.

---

## Exercice 6 · Faire une revue croisée et transmettre le résultat

**Durée : 15 minutes**

### Étape 1. Échanger les postes

Le groupe A confie au groupe B une tâche métier formulée comme un objectif, **sans indiquer où cliquer**. Le groupe B réalise le parcours et le groupe A note :

- si le parcours aboutit ;
- les hésitations et incompréhensions ;
- les erreurs de navigation ;
- les actions qui ne fonctionnent pas ;
- les questions sur les règles métier.

Puis inversez les rôles.

### Étape 2. Vérifier la remise pour le jeudi et le vendredi

- [ ] La galerie des écrans est consultable.
- [ ] Le prototype est utilisable et réinitialisable.
- [ ] Les requirements sont liés aux écrans et aux actions.
- [ ] Les questions ouvertes restent identifiées.
- [ ] Le guide de démonstration et les tâches utilisateur existent.
- [ ] Les critères d'acceptation sont documentés, sans prétendre avoir exécuté des tests.
- [ ] Le dossier `handoff/` est complet.
- [ ] Le code de production n'a pas été modifié.

### Étape 3. Préparer le passage de relais vers le développement

Le vendredi, le responsable du développement pourra ouvrir un **nouveau chat Cursor** et utiliser le contenu de `handoff/PROMPT_FOR_DEV_AGENT.md`. Le prompt doit lui demander de lire les requirements originaux, le design system et le dossier de transmission, puis de proposer un plan d'implémentation adapté à l'application existante.

Si `handoff-manifest.json` indique `review_required`, l'agent de développement doit identifier les validations encore nécessaires. Il ne doit pas supposer que le prototype a été approuvé par un sponsor ou des utilisateurs.

---

## Résultat final attendu

À la fin du lab, le projet contient au minimum :

```text
mon-application/
├── .cursor/
│   └── agents/
│       ├── design-generator.md
│       └── proto-builder.md
├── docs/
│   ├── requirements.md
│   └── design-system.md
├── design-output/
│   └── <featureId>/
│       ├── screens/index.html
│       ├── screens-manifest.json
│       ├── requirements-mapping.md
│       ├── ux-decisions.md
│       ├── interactive/index.html
│       ├── sponsor-demo.md
│       ├── user-session-guide.md
│       └── handoff/
│           ├── IMPLEMENTATION_BRIEF.md
│           ├── ACCEPTANCE_CRITERIA.md
│           ├── TRACEABILITY.md
│           ├── handoff-manifest.json
│           └── PROMPT_FOR_DEV_AGENT.md
└── ... code existant, inchangé
```

Le résultat est une **proposition UX testable**, pas une fonctionnalité déjà implémentée en production.

## En cas de problème

| Symptôme | Vérification / correction |
|---|---|
| Cursor ne reconnaît pas le sous-agent | Vérifier `.cursor/agents/<nom>.md`, la section YAML et le mode Agent. Mentionner explicitement `/design-generator` ou son nom. |
| Cursor ne trouve pas les sources | Vérifier que le bon projet est ouvert et donner les chemins exacts. |
| Les écrans ne respectent pas le design system | Demander à l'agent de citer les tokens utilisés et de corriger les écarts. |
| Le prototype ne répond pas aux clics | Contrôler les erreurs JavaScript et faire corriger l'action `ACT-*` concernée. |
| Le prototype utilise de vraies API | Stopper l'exécution et remplacer ces appels par des mocks locaux. |
| Le dossier handoff ne suffit pas pour coder | Vérifier la couverture REQ / écran / action / critère, puis compléter les points ouverts. |
| La génération prend trop de temps | Limiter la démonstration à un parcours métier prioritaire, sans perdre la traçabilité des autres exigences. |

## Pour l'animateur

**Avant :** préparer les requirements, le design system et la base de code ; choisir une fonctionnalité réalisable avec 3 à 5 écrans ; vérifier l'accès à Cursor et au navigateur.

**Pendant :** faire expliquer aux participants pourquoi chaque consigne de l'agent existe ; demander une revue humaine entre les deux agents ; signaler les hypothèses inventées par l'IA ; garder le dernier quart d'heure pour l'observation utilisateur et le handoff.

**Après :** récupérer le dossier `design-output/<featureId>/` et les deux agents. La qualité du résultat se juge sur la couverture des exigences, la cohérence visuelle, la capacité à réaliser les parcours et la clarté du passage de relais.

### Barème indicatif

| Critère | Poids |
|---|---:|
| Couverture des exigences et traçabilité | 25 % |
| Respect du design system | 20 % |
| Interactions du prototype | 25 % |
| Dossier de transmission | 20 % |
| Contrôles et absence de changements en production | 10 % |

## Ressources

- [Documentation officielle Cursor : Subagents](https://cursor.com/docs/subagents)
- Corrigé 1 : `solutions/.cursor/agents/design-generator.md`
- Corrigé 2 : `solutions/.cursor/agents/proto-builder.md`