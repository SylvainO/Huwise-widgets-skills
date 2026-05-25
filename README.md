# huwise-widget-skills

Skills Claude réutilisables pour l'écosystème **Huwise** (ex-OpenDataSoft).

Ces skills sont des fichiers Markdown structurés qui instruisent Claude sur les patterns, pièges et exemples de code spécifiques à la plateforme Huwise. Ils s'adressent aux développeurs et intégrateurs de portails de données publiques françaises.

---

## Ce qu'est ce repo

Huwise (anciennement OpenDataSoft) est une plateforme de portails open data. Le développement de pages et de composants repose sur une stack **HTML/CSS + AngularJS** (directives ODS widgets) avec des contraintes spécifiques : pas de contrôleur AngularJS, pas de JavaScript dans le header/footer, code éditeur intégré au back-office.

Ces skills permettent à Claude de connaître précisément cette stack — widgets ODS, patterns de filtrage, tableaux CSS Grid, configuration du header/footer, processeurs de données — sans avoir à réexpliquer le contexte à chaque conversation.

---

## Ressources de base

La documentation officielle des widgets ODS à consulter en priorité :

- **GitHub widgets ODS** : https://github.com/opendatasoft/ods-widgets
- **Documentation des widgets** : https://help.opendatasoft.com/widgets/#/introduction/

---

## Skills disponibles

| Skill | Description |
|---|---|
| [huwise-ods-widgets](./huwise-ods-widgets.md) | Développement de pages et dashboards avec les widgets ODS en AngularJS : `ods-dataset-context`, `ods-chart`, `ods-table`, `ods-map`, `ods-select`, `ods-adv-analysis`, `ods-results`, tableaux CSS Grid, patterns AngularJS, DSFR |
| [header-widgets](./header-widgets.md) | Configuration et personnalisation du header (bandeau de navigation) : placeholders `##menu##`, `##logo##`, `ods-responsive-menu`, dropdowns custom, header DSFR, sélecteurs CSS mobile |
| [footer-widgets](./footer-widgets.md) | Configuration et personnalisation du footer (pied de page) : placeholders `##manage-cookies##`, `##legal##`, patterns ODS natif / DSFR / full custom, logos partenaires |

---

## Comment utiliser un skill

### Option 1 — Référencer le fichier raw GitHub dans le system prompt

Copier l'URL raw du fichier `.md` et l'inclure dans votre system prompt ou dans votre configuration Claude Code :

```
https://raw.githubusercontent.com/ornetti/huwise-widget-skills/main/huwise-ods-widgets.md
```

### Option 2 — Coller le contenu directement dans une conversation

Copier le contenu du `SKILL.md` choisi et le coller dans votre conversation Claude, ou l'ajouter à votre fichier `CLAUDE.md` de projet.

### Option 3 — Plugin Claude Code (local)

Dans Claude Code, placer ce repo dans votre répertoire de plugins, ou référencer les SKILL.md dans votre `CLAUDE.md` projet :

```markdown
## Contexte technique
Voir le skill Huwise pour les patterns ODS widgets.
<skill>
[Coller ici le contenu du SKILL.md]
</skill>
```

---

## Structure du repo

```
huwise-widget-skills/
├── README.md
├── huwise-ods-widgets.md   ← Widgets ODS, dashboards, cartes, tableaux
├── header-widgets.md       ← Header, navigation, menu responsive
└── footer-widgets.md       ← Footer, liens légaux, logos
```

---

## Contexte technique commun

- **Plateforme** : Huwise (anciennement OpenDataSoft)
- **Stack** : HTML/CSS + AngularJS (directives ODS widgets)
- **Pages** : créées via le code éditeur intégré (panneau HTML + panneau CSS)
- **Cible** : portails de données publiques françaises (institutions, ministères, collectivités)
- **Référentiel design** : DSFR (Design Système de l'État français)

---

## Contribuer

Les skills sont écrits en Markdown avec un frontmatter YAML (`name`, `description`). Pour en ajouter un :

1. Créer un dossier `nom-du-skill/`
2. Y placer un `SKILL.md` avec le frontmatter et le contenu de référence
3. Mettre à jour le tableau des skills dans ce README

---

## Licence

[MIT](LICENSE)
