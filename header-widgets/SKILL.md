---
name: header-widgets
description: "Configuration et personnalisation du header (bandeau de navigation) sur la plateforme Huwise (ex-OpenDataSoft). Utiliser cette skill dès que le contexte implique : header de portail Huwise/ODS, menu de navigation, barre de navigation, bandeau supérieur, logo portail, liens de navigation, menu responsive/hamburger, ods-responsive-menu, placeholders header (##menu##, ##logo##, ##brand##, ##language##, ##secondary-menu##, ##skip-to-content-link##), configuration du header dans le back-office (Portail > Style > En-tête), feuille de style header (Portail > Style > Feuille de Style), personnalisation du nav, thème header, pied de page portail (##legal##, ##manage-cookies##, ##ods-logo##), ou toute question liée au header/footer/navigation d'un portail OpenDataSoft/Huwise. Même environnement technique que huwise-ods-widgets (AngularJS, CSS, portails données publiques françaises, DSFR)."
---

# Header Huwise / ODS — Skill de référence

## 1. Contexte technique

### Plateforme
- **Huwise** (anciennement OpenDataSoft) — plateforme de données ouvertes
- Le header se configure au **niveau du domaine Huwise** (back-office), pas dans le code éditeur des pages
- Stack : **HTML/CSS + AngularJS** (directives Angular standard utilisables : `ng-if`, `ng-class`, `ng-show`, `ng-hide`)
- **JavaScript interdit** dans le header/footer pour raisons de sécurité

### Où ça se configure dans le back-office
- **HTML du header** : Portail > Style > onglet **En-tête**
- **CSS du header** : Portail > Style > onglet **Feuille de Style**
- Le header est **commun à toutes les pages** du portail
- Le thème est **versionné** : chaque modification crée un brouillon qu'il faut rendre actif via le bouton "Rendre cette version active"

### Documentation de référence
- Doc thèmes portail : https://userguide.huwise.com/fr/articles/2042818
- Doc widgets : https://help.opendatasoft.com/widgets/#/introduction/
- GitHub widgets : https://github.com/opendatasoft/ods-widgets
- Code Library : https://codelibrary.opendatasoft.com/
- User Guide Huwise : https://userguide.huwise.com/fr

---

## 2. Structure HTML par défaut du header

Voici le HTML par défaut d'un nouveau portail Huwise :

```html
##skip-to-content-link##
<nav class="ods-front-header" ods-responsive-menu breakpoint="1000">
    <ods-responsive-menu-placeholder>
        <a class="ods-front-header__portal-brand" href="/">
            ##logo##
        </a>
    </ods-responsive-menu-placeholder>
    <ods-responsive-menu-collapsible>
        ##language##
        <a class="ods-front-header__portal-brand" href="/">
            ##logo##
            ##brand##
        </a>
        ##menu##
        ##secondary-menu##
    </ods-responsive-menu-collapsible>
</nav>
```

### Composants clés

#### `ods-responsive-menu`
Directive qui gère le comportement responsive du header. L'attribut `breakpoint` définit la largeur (en px) en dessous de laquelle le menu se replie en menu hamburger latéral.

- `<ods-responsive-menu-placeholder>` : contenu visible **quand le menu est replié** (typiquement le logo seul + bouton hamburger)
- `<ods-responsive-menu-collapsible>` : contenu visible **quand le menu est déplié** (version desktop complète)

#### Placeholders disponibles dans le header

| Placeholder | Description |
|---|---|
| `##skip-to-content-link##` | Lien d'accessibilité "Aller au contenu" |
| `##menu##` | Menu principal — liens vers les pages configurées dans le back-office |
| `##secondary-menu##` | Menu secondaire — liens connexion / compte utilisateur |
| `##logo##` | Logo du portail (configuré dans Branding) |
| `##brand##` | Nom/marque du portail (configuré dans Branding) |
| `##language##` | Sélecteur de langue du portail |

#### Placeholders disponibles dans le footer

| Placeholder | Description |
|---|---|
| `##legal##` | Lien vers les Conditions Générales |
| `##manage-cookies##` | Lien gestion des cookies (obligatoire pour la bannière cookies) |
| `##language##` | Sélecteur de langue |
| `##ods-logo##` | Logo Huwise |

---

## 3. Classes CSS du header

### Convention de nommage
Les classes CSS Huwise suivent une convention BEM : `.ods-block[--blockmodifier][__element][--elementmodifier]`

### Classes principales
- `.ods-front-header` — Le conteneur `<nav>` principal du header
- `.ods-front-header__portal-brand` — Le lien logo + marque

### Personnalisation CSS
La couleur de fond du header et les couleurs des liens se définissent via les **sélecteurs de couleur** dans l'onglet Thème de portail du back-office. Pour aller plus loin, utiliser la **Feuille de style** (Portail > Style > Feuille de Style).

> Les règles CSS ajoutées dans la feuille de style ont **priorité** sur les configurations de l'onglet Thème de portail.

---

## 4. Patterns et bonnes pratiques

### Directives AngularJS utilisables
Le header est une **application AngularJS indépendante**. On peut donc utiliser :
- `ng-if` / `ng-show` / `ng-hide` — affichage conditionnel
- `ng-class` — classes dynamiques
- Mais **pas de JavaScript** ni de contrôleurs custom

### Breakpoint responsive
Le `breakpoint="1000"` par défaut peut être ajusté selon les besoins du portail (nombre d'items de menu, largeur du logo, etc.).

### Accessibilité
- Toujours conserver `##skip-to-content-link##` en premier élément pour l'accessibilité
- Les normes WCAG AA s'appliquent : ratio de contraste texte/fond ≥ 4.5:1, éléments visuels ≥ 3:1

### Workflow de modification
1. Aller dans **Portail > Style**
2. Modifier le HTML dans l'onglet **En-tête** et/ou le CSS dans **Feuille de Style**
3. Cliquer **Enregistrer**
4. Vérifier avec le bouton **Aperçu** (ouvre un nouvel onglet)
5. Quand satisfait, cliquer **Rendre cette version active**

> Attention : une fois un thème mis en ligne, il est verrouillé. Seul le brouillon en cours est modifiable.

---

## 5. Pattern : Menu dropdown custom

Le menu natif `##menu##` ne propose **pas de sous-menus dropdown**. Pour ajouter des sous-menus déroulants, il faut remplacer `##menu##` par un menu custom construit manuellement en HTML.

Deux variantes de dropdown existent dans la pratique :

### Variante A — Classes ODS (ex: CNAF)

Utilise les classes natives ODS avec ajout de classes custom pour le dropdown :

```html
<ul class="ods-front-header__menu custom-menu">
    <!-- Item simple -->
    <li class="ods-front-header__menu-item">
        <a href="/explore/" class="ods-front-header__menu-item-link">
            <span ng-if="'fr'|currentLanguage">Données</span>
        </a>
    </li>
    <!-- Item dropdown -->
    <li class="ods-front-header__menu-item dropdown-menu">
        <a href="" class="ods-front-header__menu-item-link">
            <span ng-if="'fr'|currentLanguage">Visualisations</span>
            <ul class="dropdown-menu-content">
                <a class="ods-front-header__menu-item-link" href="/pages/ma-page">
                    <span ng-if="'fr'|currentLanguage">Sous-item 1</span>
                </a>
            </ul>
        </a>
    </li>
</ul>
```

CSS clé : `.dropdown-menu:hover .dropdown-menu-content { display: block; }`

### Variante B — Classes DSFR (ex: Data ES)

Utilise les classes du Design Système de l'État, avec un pattern dropdown différent :

```html
<div class="fr-header__menu fr-modal">
    <div class="fr-container">
        <div class="fr-nav">
            <ul class="fr-nav__list">
                <!-- Item simple -->
                <li class="fr-nav__item">
                    <a class="fr-nav__link" href="/pages/accueil/">Accueil</a>
                </li>
                <!-- Item dropdown -->
                <li class="fr-nav__item">
                    <div class="dropdown">
                        <a class="dropbtn">Tableaux de bords <i class="fa fa-angle-down"></i></a>
                        <div class="dropdown-content">
                            <a href="/pages/page1/">Sous-item 1</a>
                            <a href="/pages/page2/">Sous-item 2</a>
                            <!-- Item conditionnel (utilisateurs connectés) -->
                            <a ng-if="user.is_authenticated" href="/pages/suivi/">
                                <i class="fa fa-unlock-alt" aria-hidden="true"></i> Suivi
                            </a>
                        </div>
                    </div>
                </li>
            </ul>
        </div>
    </div>
</div>
```

CSS clé : `.dropdown:hover .dropdown-content { display: block; }`

### Différences entre les deux variantes

| | Variante A (ODS) | Variante B (DSFR) |
|---|---|---|
| Classes menu | `.ods-front-header__menu-item` | `.fr-nav__item` / `.fr-nav__link` |
| Classes dropdown | `.dropdown-menu` + `.dropdown-menu-content` | `.dropdown` + `.dropbtn` + `.dropdown-content` |
| Sous-items dans | `<ul>` dans le `<a>` parent | `<div>` sibling du `<a>` parent |
| Indicateur dropdown | Aucun (texte seul) | `<i class="fa fa-angle-down">` |
| Multi-langue | `ng-if="'fr'\|currentLanguage"` sur chaque `<span>` | Texte directement dans les `<a>` |
| Polices | Thème ODS | Marianne (DSFR) |

---

## 6. Pattern : Header DSFR complet (sans placeholders natifs)

Pour les portails gouvernementaux qui veulent une conformité DSFR poussée, il est possible de **remplacer tous les placeholders natifs** par du HTML custom :

### Structure type

```html
<nav breakpoint="992" class="fr-header" ods-responsive-menu>
    <ods-responsive-menu-placeholder>
        <a href="/pages/accueil/">
            <img class="fr-header__logo"
                 src="/assets/theme_image/logo-ministere.png"
                 alt="Nom du ministère">
        </a>
    </ods-responsive-menu-placeholder>
    <ods-responsive-menu-collapsible>
        <!-- Zone haute : logo + brand + CTA -->
        <div class="fr-header__body">
            <div class="fr-container">
                <div class="fr-header__body-row">
                    <a href="/pages/accueil/" class="fr-header__brand">
                        <img class="fr-header__logo" src="/assets/theme_image/logo-ministere.png"
                             alt="Nom du ministère"/>
                        <div class="fr-header__service">
                            <p class="header-text">nom-du-portail<span style="color:#3a3a3a;">.gouv.fr</span></p>
                            <p class="header-tagline">Sous-titre descriptif</p>
                        </div>
                    </a>
                    <div class="fr-header__under-login">
                        <a class="fr-header__button" href="/pages/action/">Bouton CTA</a>
                    </div>
                </div>
            </div>
        </div>
        <!-- Zone basse : navigation -->
        <div class="fr-header__menu fr-modal">
            <div class="fr-container">
                <div class="fr-nav">
                    <ul class="fr-nav__list">
                        <!-- items ici -->
                    </ul>
                </div>
            </div>
        </div>
    </ods-responsive-menu-collapsible>
</nav>
```

### Points clés de ce pattern
- **Aucun placeholder** (`##logo##`, `##brand##`, `##menu##`, `##secondary-menu##`) — tout est en HTML custom
- Logo uploadé dans **Portail > Images et polices** → URL `/assets/theme_image/nom-fichier.png`
- Le header se divise en **deux zones** : `.fr-header__body` (logo/brand/CTA) et `.fr-header__menu` (nav)
- Un **CTA** (bouton d'action) peut être positionné en absolu à droite du header
- Le `##secondary-menu##` peut être **commenté** si on ne veut pas du menu de connexion natif
- Variables DSFR utilisées : `--blue-france-sun-113-625`, `--blue-france-975-75`, `--text`, `--foot-border`

### CSS mobile spécifique (sélecteurs longs)

Quand on utilise des classes DSFR custom à l'intérieur de `ods-responsive-menu`, le CSS mobile nécessite des **sélecteurs très spécifiques** traversant toute la hiérarchie DOM générée par la directive :

```css
/* Items en pleine largeur en mobile */
.ods-responsive-menu-collapsible--collapsed > .ods-responsive-menu-collapsible__container
  > .ods-responsive-menu-collapsible__content > .fr-header__menu > .fr-container
  > .fr-nav > .fr-nav__list > .fr-nav__item {
    width: 100%;
    margin-left: 0;
}

/* Dropdown en position relative en mobile */
.ods-responsive-menu-collapsible--collapsed > .ods-responsive-menu-collapsible__container
  > .ods-responsive-menu-collapsible__content > .fr-header__menu > .fr-container
  > .fr-nav > .fr-nav__list > .fr-nav__item > .dropdown > .dropdown-content {
    position: relative;
    width: 100%;
}

/* Masquer le logo/brand dans le menu mobile */
.ods-responsive-menu-collapsible--collapsed > .ods-responsive-menu-collapsible__container
  > .ods-responsive-menu-collapsible__content > .fr-header__body {
    display: none;
}
```

> Ces sélecteurs longs sont nécessaires car on ne contrôle pas les `<div>` intermédiaires générés par `ods-responsive-menu`. Il faut traverser toute la hiérarchie pour cibler les bons éléments.

---

## 7. Pattern : Brand layout custom avec menu natif

Pour les portails qui veulent un layout de header personnalisé (logo + titre côte à côte) tout en conservant le `##menu##` natif, il est possible de créer un wrapper custom autour du brand :

```html
<nav class="ods-front-header" ods-responsive-menu breakpoint="1000">
    <ods-responsive-menu-placeholder>
        <a class="ods-front-header__portal-brand" href="/">
            ##logo##
        </a>
    </ods-responsive-menu-placeholder>
    <ods-responsive-menu-collapsible>
        <div class="ods-front-header__brand-block">
            <div style="display: flex;">
                <div class="logo-header">
                    <a class="ods-front-header__portal-brand" href="/">
                        <img src="/assets/theme_image/mon-logo.png">
                    </a>
                </div>
                <div class="header-body">
                    <div class="header-title">MON TITRE CUSTOM</div>
                    <div class="ods-front-header__menu-block">
                        ##menu##
                    </div>
                </div>
            </div>
        </div>
        ##secondary-menu##
    </ods-responsive-menu-collapsible>
</nav>
```

### Points clés
- Le `##menu##` natif est conservé → les items de menu restent configurables dans le back-office
- La classe `ods-front-header__menu-item--active` est ajoutée automatiquement par la plateforme sur la page courante
- Le titre est en HTML custom (pas `##brand##`) pour plus de contrôle sur le style
- Le layout flexbox du wrapper positionne le logo à gauche et le titre + menu à droite
- Pas de CSS custom nécessaire si les styles par défaut ODS conviennent

---

## 8. Directives AngularJS utiles dans le header

| Directive / Expression | Usage | Exemple |
|---|---|---|
| `ng-if="'fr'\|currentLanguage"` | Afficher selon la langue active | Multi-langue |
| `ng-if="user.is_authenticated"` | Visible uniquement si connecté | Lien admin |
| `ng-class` | Classes dynamiques | Style actif sur la page courante |
| `ng-show` / `ng-hide` | Afficher/masquer (garde le DOM) | Toggle d'éléments |

---

## 9. Exemples concrets

Pour des exemples complets avec HTML back-office, HTML rendu et CSS, consulter :
→ `references/examples.md`

Exemples disponibles :
1. **CNAF (data.caf.fr)** — Menu custom ODS avec dropdowns, classes `.ods-front-header__*`
2. **Équipements sportifs (equipements.sports.gouv.fr)** — Header DSFR complet, classes `.fr-*`, CTA, items conditionnels
3. **CNAV (cnav.huwise.com)** — Menu natif `##menu##` + layout brand custom avec flexbox

---

## 10. Checklist header

- [ ] Le header est configuré dans **Portail > Style > En-tête** (pas dans le code éditeur des pages)
- [ ] Le CSS du header est dans **Portail > Style > Feuille de Style**
- [ ] `##skip-to-content-link##` est en premier pour l'accessibilité
- [ ] Le `breakpoint` de `ods-responsive-menu` est adapté au contenu du menu
- [ ] Pas de JavaScript dans le header (interdit par la plateforme)
- [ ] Le thème a été prévisualisé avant mise en ligne
- [ ] Les contrastes de couleurs respectent WCAG AA
- [ ] Le breakpoint des media queries CSS correspond au `breakpoint` de `ods-responsive-menu`
- [ ] Les dropdowns ont un `z-index` suffisant (≥ 20) pour passer au-dessus du contenu
- [ ] Le `background-color` du dropdown est explicite (sinon transparent au hover)
- [ ] Les liens externes ont `target="_blank"`
- [ ] Le menu mobile repasse les dropdowns en `position: relative`
- [ ] Si header DSFR : les sélecteurs CSS mobile traversent la hiérarchie complète de `ods-responsive-menu`
- [ ] Si logo custom : image uploadée dans Portail > Images et polices, référencée via `/assets/theme_image/`

---
