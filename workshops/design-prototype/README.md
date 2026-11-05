# Atelier 3 : Design & Prototype dans Cursor

Ouvrir `index.html` dans un navigateur pour suivre le lab interactif, ou `LAB.md` pour la version Markdown.

Les deux agents de référence se trouvent dans `solutions/.cursor/agents/`. Ne les distribuez qu'après la phase de conception si l'objectif est pédagogique.

**Format :** 2 h 15, dont 90 minutes de pratique.  
**Entrées :** requirements Markdown, design system Markdown existant et application existante.  
**Sortie :** maquettes, prototype interactif et dossier de transmission au développement.

## Publication sur Vercel

Ce dossier contient un site statique (aucun build nécessaire).

1. Importer le dépôt GitHub `yassineaqejjaj/agentic-sdlc` dans Vercel.
2. Définir **Root Directory** sur `workshops/design-prototype`.
3. Choisir **Framework Preset: Other**.
4. Ne pas définir de build command ; conserver `index.html` comme point d'entrée.
5. Lancer **Deploy**, puis vérifier la page et les liens.
6. Activer les déploiements Git pour que les changements de la branche `main` soient publiés.

Le fichier `vercel.json` est inclus dans ce dossier.
