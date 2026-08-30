# CLAUDE.md

Guide pour Claude Code (claude.ai/code) sur ce dépôt.

## État du dépôt

`trading-Alice` est actuellement **vide** — aucun code, aucune stack choisie. Les sections
« Architecture », « Commandes » et « Conventions » ci-dessous sont des emplacements à remplir
au fur et à mesure. Ne pas inventer de structure : si une information manque ici, la demander
ou la déduire du code réellement présent.

## Architecture

_À compléter quand le premier code arrive._

## Commandes

_À compléter : build, tests, lint, lancement local._

## Conventions

_À compléter : style, nommage, structure des dossiers, format des commits._

## Configuration Claude Code

### Permissions (`.claude/settings.json`)

Le fichier est volontairement **restrictif**. Trois listes, évaluées dans l'ordre
`deny` → `ask` → `allow` (la première règle qui correspond gagne) :

- **`deny`** — lecture des secrets (`.env`, `secrets/`, `*.pem`, `~/.ssh`, `~/.aws`) et
  `curl`/`wget`. Le blocage réseau côté shell est délibéré : il ferme le vecteur
  « télécharger un script et l'exécuter ». Pour consulter le web, passer par `WebFetch`.
- **`ask`** — tout ce qui est destructif, sortant, ou qui installe du code distant :
  `git push`, `rm`, `sudo`, installations de paquets, `docker`, `terraform`, cloud CLIs.
  Y figurent aussi les fichiers qui **régissent les permissions elles-mêmes**
  (`.claude/settings.json`, `.claude/hooks/`, `.mcp.json`, `.github/workflows/`) : Claude ne
  peut pas élargir ses propres droits sans validation explicite.
- **`allow`** — lecture/édition des fichiers du projet, les sous-commandes git non
  destructives, les commandes shell en lecture seule, et les lanceurs de tests/linters.

Points d'attention si vous modifiez ce fichier :

- Dans une règle `allow`, `Write(chemin)` et `NotebookEdit(chemin)` sont acceptés au parsing
  mais **jamais consultés** — seuls `Edit(chemin)` et `Read(chemin)` comptent. Utiliser
  `Edit(...)`. Dans `deny`/`ask`, `Write(...)` fonctionne normalement.
- Une commande composée (`&&`, `||`, `;`, `|`) est découpée : **chaque** sous-commande doit
  correspondre à une règle. `Bash(pytest *)` n'autorise donc pas `pytest && rm -rf .`.
- `Bash(ls *)` (avec espace) matche `ls -la` mais pas `lsof`. Sans espace, `Bash(ls*)` matche
  les deux.

Élargir la liste `allow` au fur et à mesure que la stack se précise (interpréteur, runner de
tests réel, gestionnaire de paquets) plutôt que de repartir sur un `Bash(*)` global.

### Réglages personnels

Les préférences individuelles vont dans `.claude/settings.local.json` (ignoré par git), jamais
dans le fichier partagé. Ordre de précédence : managed → arguments CLI → `settings.local.json`
→ `settings.json` → `~/.claude/settings.json`.

### Ce qui n'est pas configuré ici

Pas de `.mcp.json`, pas de hooks, pas de sous-agents. Ces éléments seront ajoutés quand un
besoin concret apparaîtra — et un sous-agent qui traite du contenu externe (dépôts tiers, pages
web, issues) ne doit **jamais** recevoir `permissionMode: bypassPermissions` : ce serait un
canal direct entre du texte non fiable et l'exécution de commandes sans garde-fou.

## Bonnes pratiques de travail

- Garder ce fichier sous ~200 lignes : au-delà, l'adhérence se dégrade.
- Démarrer les tâches complexes en mode plan.
- Découper les sous-tâches pour qu'elles tiennent en moins de 50 % du contexte ;
  `/compact` manuel autour de 50 %.
- Lancer les commandes longues en tâche de fond pour garder les logs lisibles.
- `/doctor` pour diagnostiquer un problème de configuration.
