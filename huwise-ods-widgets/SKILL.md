---
name: huwise-ods-widgets
description: "Développement de pages et dashboards sur la plateforme Huwise (ex-OpenDataSoft) avec les widgets ODS en AngularJS. Utiliser cette skill dès que le contexte implique : widgets ODS (ods-dataset-context, ods-chart, ods-table, ods-map, ods-select, ods-simple-tabs, ods-adv-analysis, ods-results, ods-facets, etc.), code éditeur Huwise, portails de données publiques françaises, CSS pour portails ODS, Design Système d'État (DSFR), tableaux en CSS Grid avec colonnes sticky, filtrage de datasets, ou toute question liée à l'écosystème OpenDataSoft/Huwise."
---

# Huwise / ODS Widgets — Skill de référence

## 1. Contexte technique

### Plateforme
- **Huwise** (anciennement OpenDataSoft) — plateforme de données ouvertes
- Pages créées via le **code éditeur** intégré (HTML + CSS)
- Stack : **HTML/CSS + AngularJS** (via les directives ODS widgets)

### Documentation de référence
- Doc widgets : https://help.opendatasoft.com/widgets/#/introduction/
- GitHub widgets : https://github.com/opendatasoft/ods-widgets
- Code Library (exemples) : https://codelibrary.opendatasoft.com/
- User Guide Huwise : https://userguide.huwise.com/fr
- Documentation des widgets : https://help.opendatasoft.com/widgets/#/introduction/
- Et notamment ODS Map : https://help.opendatasoft.com/widgets/#/api/ods-widgets.directive:odsMap

### Contraintes de l'environnement
- **Pas de contrôleur AngularJS** — tout doit rester dans le HTML via des directives (`ng-if`, `ng-repeat`, `ng-init`, `ng-click`, `ng-switch`, `ng-class`, etc.)
- **Pas de fonctions anonymes** dans les expressions `ng-click` ou les templates (AngularJS les interdit dans les expressions)
- Le CSS est écrit dans un panneau séparé du code éditeur
- Les portails ont souvent un thème CSS pré-existant (variables CSS du portail)

---

## 2. Widgets principaux et patterns

### 2.1 `ods-dataset-context` — Le fondement

Déclare un ou plusieurs contextes de données. Chaque contexte est lié à un dataset et peut recevoir des paramètres de filtrage.

```html
<ods-dataset-context context="ctx,ctxfilter"
                     ctx-dataset="mon-dataset"
                     ctx-domain="mon-domaine.opendatasoft.com"
                     ctx-parameters="{'refine.champ':'valeur', 'disjunctive.champ':true, 'sort':'datenotification'}"
                     ctxfilter-dataset="mon-dataset"
                     ctxfilter-domain="mon-domaine.opendatasoft.com"
                     ctx-urlsync="true">
```

**Paramètres clés :**
- `refine.champ` : filtre sur une valeur précise
- `disjunctive.champ` : active le mode "OU" pour le multi-refine
- `sort` : tri (préfixer de `-` pour tri descendant)
- `urlsync` : synchronise les filtres avec l'URL

**Pattern important — Contextes séparés pour filtres et listes :**
Quand un `ods-select` ou un `ods-adv-analysis` alimente une liste d'options ET qu'un refine s'applique au même contexte, la liste se réduit après sélection. Solution : utiliser **deux contextes séparés** — un pour la liste (jamais raffiné), un pour les visualisations (raffiné).

```html
<ods-dataset-context context="ctx,ctxpourliste"
                     ctx-dataset="dataset"
                     ctx-parameters="{'disjunctive.age':true}"
                     ctxpourliste-dataset="dataset">
```

### 2.2 `ods-adv-analysis` — Agrégations avancées

Permet des requêtes d'agrégation (count, sum, group by, where, order by).

```html
<div ods-adv-analysis="resultats"
     ods-adv-analysis-context="ctx"
     ods-adv-analysis-select="count(*) as total, sum(montant) as montant"
     ods-adv-analysis-where="champ LIKE 'valeur'"
     ods-adv-analysis-group-by="champ1,champ2"
     ods-adv-analysis-order-by="champ1"
     ods-adv-analysis-limit="50">
    {{ resultats }}
</div>
```

**Pattern — Stocker dans une variable intermédiaire :**
```html
<div ods-adv-analysis="ages"
     ods-adv-analysis-context="ctxpourliste"
     ods-adv-analysis-select="count(*)"
     ods-adv-analysis-group-by="age"
     ods-adv-analysis-order-by="age"
     ods-adv-analysis-limit="50">
    {{ values.ages = ages; "" }}
</div>
```

Le `; ""` à la fin empêche l'affichage de la valeur dans le DOM.

**Pattern — Construction de requête dynamique avec ng-repeat :**
```html
<div ng-repeat="item in items">
    {{ map.query = $index === 0
       ? 'champ:' + item.id
       : map.query + ' OR champ:' + item.id; "" }}
</div>
```

### 2.3 `ods-select` — Filtres dropdown

```html
<ods-select ng-init="selected.ages = [{'name':'Ensemble'}]"
            selected-values="selected.ages"
            placeholder="Sélectionnez..."
            label-modifier="age"
            value-modifier="{'name':age}"
            options="values.ages"
            multiple="true"
            is-loading="true">
</ods-select>

{{ ctx.parameters['refine.age'] = (selected.ages | toObject:'name' | keys); "" }}
```

**Paramètres :**
- `options` : tableau d'objets source
- `selected-values` : variable two-way binding pour la sélection
- `label-modifier` : champ affiché
- `value-modifier` : format de la valeur stockée
- `multiple` : active le multi-select
- `is-loading` : affiche un indicateur de chargement
- `placeholder`, `disabled`, `on-change`

**Piège classique :** Si le `ods-adv-analysis` qui alimente les options utilise le même contexte que celui qui reçoit le refine, la liste d'options se réduit après sélection. → Utiliser deux contextes séparés (voir section 2.1).

**Initialisation par défaut avec ng-init :**
Le `ng-init` sur le `ods-select` doit utiliser le même format objet que le `value-modifier`.

### 2.4 `ods-chart` — Graphiques

```html
<ods-chart timescale="year" single-y-axis="true" align-month="true">
    <ods-chart-query context="ctx" field-x="annee" maxpoints="0" timescale="year">
        <ods-chart-serie expression-y="mon_champ"
                         chart-type="line"
                         function-y="MAX"
                         label-y="Ma série"
                         color="#1b1b35"
                         scientific-display="true">
        </ods-chart-serie>
    </ods-chart-query>
</ods-chart>
```

**Pattern — Séries dynamiques avec ng-if :**
Quand on veut afficher/masquer des séries selon la sélection d'un filtre multi-valeurs :

```html
<ods-chart-query ng-if="ctx.parameters['refine.age'].indexOf('Ensemble') > -1"
                 context="ctxensemble"
                 field-x="annee" maxpoints="0" timescale="year">
    <ods-chart-serie expression-y="pourcentage"
                     chart-type="line" function-y="MAX"
                     label-y="Ensemble" color="#1b1b35">
    </ods-chart-serie>
</ods-chart-query>
```

Chaque valeur de filtre a son propre contexte pré-raffiné et son `ods-chart-query` conditionnel.

**Pattern — Forcer le re-render avec ng-switch :**
```html
{{ chartKey = ctx.parameters['refine.age'].join(','); "" }}
<div ng-switch="chartKey">
    <div ng-switch-default>
        <ods-chart>...</ods-chart>
    </div>
</div>
```

Utiliser des variables `chartKey` distinctes si plusieurs sections de charts coexistent (ex. `chartKey`, `chartKeyGeneral`, `chartKeyTechno`, `chartKeyPro`).

### 2.5 `ods-results` — Affichage de résultats

```html
<div ods-results="lines" ods-results-context="ctx" ods-results-max="15">
    <div ng-repeat="line in lines track by $index">
        {{ line.fields.mon_champ }}
    </div>
</div>
```

Le `track by $index` optimise le rendu AngularJS en réutilisant les éléments DOM existants.

### 2.6 `ods-simple-tabs` — Navigation par onglets

```html
<section class="pills-container">
    <ods-simple-tabs class="tab-pills">
        <ods-simple-tab label="Onglet 1"
                        fontawesome-class="building"
                        keep-content="false">
            <!-- Contenu onglet 1 -->
        </ods-simple-tab>
        <ods-simple-tab label="Onglet 2"
                        fontawesome-class="battery-full"
                        keep-content="false">
            <!-- Contenu onglet 2 -->
        </ods-simple-tab>
    </ods-simple-tabs>
</section>
```

**CSS pour centrer les pills et couleurs par onglet :**
```css
.pills-container {
    display: flex;
    justify-content: center;
    align-items: center;
}
.tab-pills { width: 100%; }
.tab-pills ul {
    display: flex;
    justify-content: center;
    width: 100%;
}
.tab-pills .ods-simple-tabs-nav-link {
    padding: 0.5rem 1.5rem;
    border-bottom: 0;
    border-radius: 30px;
    background-color: transparent;
    font-size: 1.5em;
}
/* Couleur par onglet — utiliser li:nth-child car les <a> sont enfants de <li> */
.tab-pills li:first-child .ods-simple-tabs-nav-link:hover:not(.ods-simple-tabs-nav-link-active) {
    color: #1a2783;
}
.tab-pills li:first-child .ods-simple-tabs-nav-link.ods-simple-tabs-nav-link-active {
    background-color: #1a2783;
    color: #FFFFFF;
}
.tab-pills li:nth-child(2) .ods-simple-tabs-nav-link:hover:not(.ods-simple-tabs-nav-link-active) {
    color: #F39200;
}
.tab-pills li:nth-child(2) .ods-simple-tabs-nav-link.ods-simple-tabs-nav-link-active {
    background-color: #F39200;
    color: #FFFFFF;
}
```

**Piège :** Les sélecteurs `:first-child` / `:nth-child(2)` doivent cibler les `<li>` parents, pas directement les `.ods-simple-tabs-nav-link` (car chaque lien est un enfant unique de son `<li>`, donc tous sont `:first-child`).

### 2.7 `ods-map` — Cartes choroplèthes

Structure de base pour une carte choroplèthe par département :

```html
<ods-map style="height: 700px"
         no-refit="true"
         scroll-wheel-zoom="false"
         display-control="false"
         search-box="false"
         toolbar-fullscreen="true"
         toolbar-geolocation="false"
         basemap="ign.planv2"
         location="6,46.5,2.5">
    <ods-map-layer-group>
        <ods-map-layer context="moncontexte"
                       color-numeric-ranges="{'99.5':'#D2EDFB','102.4':'#A9D2F0','106.1':'#85B1DF','109.3':'#6093CD','129.6':'#3F74B8'}"
                       color-numeric-range-min="69.8"
                       show-marker="true"
                       display="choropleth"
                       function="AVG"
                       expression="mon_champ_numerique"
                       shape-opacity="0.85"
                       point-opacity="1"
                       border-color="#FFFFFF"
                       border-opacity="1"
                       border-size="1"
                       border-pattern="solid"
                       caption="false"
                       title="Mon titre"
                       size="4">
        </ods-map-layer>
    </ods-map-layer-group>
</ods-map>
```

**Paramètres clés de `ods-map` :**
- `basemap` : fond de carte — `ign.planv2`, `ign.parcellaire-express`, `jawg.light`, etc.
- `location` : `"zoom,lat,lng"` — **le zoom DOIT être un entier** (voir piège ci-dessous)
- `no-refit` : empêche le recadrage automatique sur les données
- `display-control` : affiche/masque le sélecteur de couches
- `scroll-wheel-zoom` : active/désactive le zoom à la molette

**Paramètres clés de `ods-map-layer` :**
- `display="choropleth"` : mode choroplèthe (coloration de shapes)
- `color-numeric-ranges` : bornes de couleur, format `{'borne_haute':'#couleur'}` — utiliser le **point** comme séparateur décimal
- `color-numeric-range-min` : borne basse de la première catégorie
- `function` : fonction d'agrégation (`AVG`, `SUM`, `COUNT`, `MIN`, `MAX`)
- `expression` : champ numérique à agréger
- `caption="false"` : désactive la légende native ODS (pour utiliser une légende custom en dur)
- `show-if` : condition AngularJS pour afficher/masquer une couche dynamiquement
- `refine-on-click-context` : permet de raffiner un autre contexte au clic sur la carte

**Piège critique — Zoom entier obligatoire :**
L'attribut `location="zoom,lat,lng"` exige un **zoom strictement entier**. Une valeur float comme `5.5` provoque l'erreur :
```
"error": "Value of parameter 'clusterprecision' is not a valid value of type integer"
```
Solution : toujours utiliser un entier (`5`, `6`, `7`...). Pour ajuster finement le cadrage, jouer sur les coordonnées lat/lng plutôt que sur le zoom. Par exemple, pour la France métropolitaine avec légende en bas : `location="6,46.8,2.5"` (décaler le centre vers le nord).

**Fonds de carte recommandés pour les portails éducation :**
- `ign.planv2` : fond IGN Plan — sobre, adapté aux choroplèthes administratives
- `ign.parcellaire-express` : fond cadastral très léger
- `jawg.light` : fond clair minimaliste

### 2.8 Légende custom inlined pour `ods-map`

Quand `caption="false"` sur le `ods-map-layer`, on peut ajouter une légende en dur en HTML, positionnée en bas de la carte. Source : [Code Library — custom legend simple inlined](https://codelibrary.opendatasoft.com/widget-tricks/ods-map-css/#custom-legend---simple-inlined).

**Structure HTML :** La légende est un sibling de `<ods-map>`, à l'intérieur d'un `<div class="map-container">` positionné en relative.

```html
<div class="map-container">
    <ods-map style="height: 700px" ...>
        <!-- layers -->
    </ods-map>

    <!-- Légende en dur -->
    <div class="odswidget odswidget-map-legend">
        <div class="odswidget-map-legend__header">
            <div class="odswidget-map-legend__title">Mon titre</div>
            <div class="odswidget-map-legend__label">Sous-titre</div>
        </div>
        <div>
            <div class="odswidget-map-legend__choropleth-container">
                <!-- Répéter pour chaque catégorie -->
                <div class="odswidget-map-legend__choropleth__item">
                    <div class="odswidget-map-legend__choropleth__item-color">
                        <div class="odswidget-map-legend__choropleth__color-block" style="background-color: #D2EDFB;"></div>
                    </div>
                    <div class="odswidget-map-legend__choropleth__item-range">
                        <div class="odswidget-map-legend__choropleth__item-range__bound">69,8
                            <i aria-hidden="true" class="fa fa-long-arrow-right odswidget-map-legend__choropleth__item-range__bound-arrow"></i>
                        </div>
                        <div class="odswidget-map-legend__choropleth__item-range__bound">99,5</div>
                    </div>
                </div>
                <!-- ... autres items ... -->
            </div>
        </div>
    </div>
</div>
```

**CSS pour la légende inlined :**
```css
.map-container {
    position: relative;
}
.odswidget-map-legend__choropleth-container {
    display: flex;
    justify-content: space-evenly;
}
.odswidget.odswidget-map-legend {
    left: 4px;
    right: 4px;
    width: inherit;
    padding: 5px 5px 0 5px;
    background-color: #ffffffd1;
}
.odswidget-map-legend__choropleth__item {
    flex-direction: column;
    align-items: center;
}
.odswidget-map-legend__choropleth__item-color {
    margin: 0;
}
.odswidget-map-legend__choropleth__color-block {
    width: 120px;
}
i.fa.fa-long-arrow-right.odswidget-map-legend__choropleth__item-range__bound-arrow {
    margin: 10px;
}
.odswidget-map-legend__choropleth__item-range__bound {
    width: inherit;
}
.odswidget-map-legend__choropleth__item-range {
    font-size: inherit;
    align-items: center;
}
/* Remonter les contrôles Leaflet au-dessus de la légende */
.leaflet-bottom.leaflet-left {
    bottom: 130px;
}
```

**Points importants :**
- Les valeurs dans `color-numeric-ranges` utilisent le **point** décimal (format API)
- Les valeurs affichées dans la légende HTML utilisent la **virgule** française (format visuel)
- La légende doit être un sibling de `<ods-map>`, pas un enfant (sinon absorbée par le widget)
- Augmenter la `height` de la carte (700px+) pour compenser l'espace pris par la légende en bas

---

## 3. Tableaux en CSS Grid

### Structure de base
```html
<div class="ge-array array-container" ods-results="lines" ods-results-context="ctx" ods-results-max="15">
    <div class="ge-array array-head"
         style="grid-template-columns: 200px 150px repeat(12, 1fr); gap:24px; position:sticky; top:0;">
        <span>Colonne 1</span>
        <span>Colonne 2</span>
        <!-- ... -->
    </div>
    <div class="ge-array array-line"
         style="grid-template-columns: 200px 150px repeat(12, 1fr); gap:24px;"
         ng-repeat="line in lines track by $index">
        <a href="/pages/fiche?id={{line.fields.id}}">{{ line.fields.nom }}</a>
        <span>{{ line.fields.valeur }}</span>
        <!-- ... -->
    </div>
</div>
```

### Colonnes sticky (2 premières colonnes)

Pour les tableaux larges avec scroll horizontal, rendre les 2 premières colonnes sticky :

```css
/* Header sticky vertical */
.ge-array .array-head {
    position: sticky;
    top: 0;
    z-index: 12;
}
/* 1ère colonne sticky horizontal — header */
.ge-array .array-head span:first-child {
    position: sticky;
    left: 0;
    z-index: 11;
    background: #E6E5F0;
    padding-right: 24px;
}
/* 2ème colonne sticky — header (left = largeur 1ère col + gap) */
.ge-array .array-head span:nth-child(2) {
    position: sticky;
    left: calc(200px + 24px);
    z-index: 11;
    background: #E6E5F0;
}
/* 1ère colonne sticky — lignes */
.ge-array .array-line a:first-child {
    position: sticky;
    left: 0;
    z-index: 10;
    background: #FFF;
    padding-right: 24px;
}
/* 2ème colonne sticky — lignes */
.ge-array .array-line a:nth-child(2),
.ge-array .array-line span:nth-child(2) {
    position: sticky;
    left: calc(200px + 24px);
    z-index: 10;
    background: #FFF;
}
```

**Points critiques :**
- Le `left` de la 2ème colonne = largeur 1ère colonne + gap
- Définir les colonnes en px fixe (pas `1fr`) pour les colonnes sticky
- Le `background` est obligatoire sinon on voit à travers au scroll
- Le `z-index` du header doit être supérieur à celui des lignes
- Ajouter un `::after` pseudo-élément pour couvrir le gap entre les colonnes sticky

### Tri dans les colonnes
```html
<span style="display:flex">
    Date d'attribution
    <span style="display:flex; gap:10px; align-items:center; flex-direction:column;">
        <i style="cursor:pointer; {{ ctx.parameters['sort']=='-datenotification' ? 'font-weight:900' : '' }}"
           ng-click="ctx.parameters['sort']='-datenotification'"
           class="fa fa-chevron-up"/>
        <i style="cursor:pointer; {{ ctx.parameters['sort']=='datenotification' ? 'font-weight:900' : '' }}"
           ng-click="ctx.parameters['sort']='datenotification'"
           class="fa fa-chevron-down"/>
    </span>
</span>
```

---

## 4. Patterns AngularJS récurrents

### Éviter le flash de contenu au chargement
Quand des données async conditionnent l'affichage, vérifier `!== undefined` pour ne rien afficher pendant le chargement :

```html
<div ng-if="conf !== undefined && conf.length > 0 && conf[0].fields.role == 'Admin'">
    <!-- Contenu réservé -->
</div>
<div ng-if="conf !== undefined && (conf.length == 0 || conf[0].fields.role != 'Admin')">
    <!-- Message d'accès restreint -->
</div>
```

### Éviter l'erreur infinite digest loop
Ne jamais modifier une variable dans une expression d'interpolation `{{ }}` qui serait réévaluée à chaque cycle. Utiliser `ng-init` pour les initialisations.

```html
<!-- MAUVAIS — provoque un infinite digest -->
{{ params.selection = (params.other.length > 0 ? params.selection : ['défaut']); '' }}

<!-- BON — initialisation dans ng-init -->
<div ng-init="params.selection = params.selection || ['défaut']">
```

### Formatage des nombres
```html
{{ (line.fields.montant | number:0) || 'Pas de données' }}
```
Les parenthèses autour du pipe `number` sont obligatoires avant le `||`.

Caractère euro : utiliser `&nbsp;€` pour un espace insécable.

### Filtre momentadd (dates)
```html
{{ variables.date | momentadd : 'years' : -1 }}
```
La syntaxe ODS est `period` en premier, `number` en second. Un warning Moment.js peut apparaître — c'est un bug interne ODS, pas dans votre code.

---

## 5. CSS — Bonnes pratiques pour portails ODS

### Variables CSS du portail
Les thèmes ODS exposent des variables CSS :
```css
:root {
    --highlight: #142E7B;
    --text: #333333;
    --boxes-border: #DEE5EF;
    --boxes-background: #FFFFFF;
    --page-background: #F6F8FB;
}
```

### Classes utilitaires courantes
```css
.d-flex { display: flex; }
.d-grid { display: grid; }
.gap-column-medium { gap: 16px; }
.mx-auto { margin-left: auto; margin-right: auto; }
.maxwidth-large { max-width: 1200px; }
.fw-bold { font-weight: bold; }
.text-small { font-size: 0.85rem; }
.color-primary { color: var(--highlight); }
```

### Conformité DSFR (Design Système de l'État)
Pour les portails gouvernementaux français :
- Police Marianne obligatoire
- Classes préfixées `fr-` (fr-container, fr-grid-row, fr-card, etc.)
- Tokens couleur officiels : bleu France `#000091`, etc.
- Système d'espacement basé sur multiples de 8px

### ODS Layout over-ride — Flexbox pour `.row`

Le layout par défaut ODS (basé sur Bootstrap 3) ne gère pas l'equal-height ni le flex-wrap. Pour les pages avec des KPI, indicateurs ou cartes côte à côte, appliquer le pattern **ODS Layout over-ride** de la [Code Library](https://codelibrary.opendatasoft.com/page-templates/ods-layout-over-ride/) :

```css
.row {
    display: flex;
    flex-wrap: wrap;
}
.row > * {
    margin-bottom: 20px;
    flex-grow: 1;
}
@media screen and (max-width: 749px) {
    .row {
        flex-direction: column;
    }
}
```

**Quand l'utiliser :**
- Dès qu'on a des `col-md-6` ou `col-md-4` qui doivent être côte à côte avec la même hauteur
- Pour les sections d'indicateurs/KPI en grille
- Pour les layouts `section > div.col-md-12 > div.card` wrappés dans une `.row`

**Piège :** Sans ce CSS, les `.row` n'appliquent pas `flex-wrap` et les colonnes peuvent se chevaucher ou ne pas se disposer correctement. Ce pattern est **essentiel** dès qu'on utilise des `.col-md-*` dans le code éditeur.

---

## 6. Sortie attendue

Quand on génère du code pour Huwise, fournir :
1. **Un bloc HTML** à coller dans le panneau HTML du code éditeur
2. **Un bloc CSS** à coller dans le panneau CSS du code éditeur
3. Les deux doivent être indépendants et complets

---

## 7. Checklist avant livraison

- [ ] Tous les contextes nécessaires sont déclarés dans le `ods-dataset-context`
- [ ] Les contextes pour listes d'options sont séparés des contextes filtrés
- [ ] Le `disjunctive` est activé sur les champs multi-refine
- [ ] Les `ng-repeat` utilisent `track by $index`
- [ ] Les expressions `{{ }}` ne modifient pas de variables (utiliser `ng-init` à la place)
- [ ] Les colonnes sticky ont un `background` explicite
- [ ] Le `z-index` du header est supérieur à celui des lignes
- [ ] Le CSS utilise les variables du portail quand elles existent
- [ ] Les nombres sont formatés avec le pipe `number` et des parenthèses
- [ ] Le CSS `.row` flex-wrap est présent si on utilise des `col-md-*`
- [ ] Le zoom dans `location` de `ods-map` est un **entier** (pas de float)
- [ ] Les `color-numeric-ranges` utilisent le **point** décimal (pas la virgule)
- [ ] Les légendes custom sont des siblings de `<ods-map>`, pas des enfants
- [ ] Les sections dataviz avec indicateurs ou cartes sont wrappées dans `<div class="card z-depth-1">` pour l'effet box-shadow