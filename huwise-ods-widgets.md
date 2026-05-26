---
name: huwise-ods-widgets
description: >
  This skill should be used when developing pages or dashboards on the Huwise (OpenDataSoft) platform
  with ODS widgets and AngularJS directives — including ods-dataset-context, ods-adv-analysis,
  ods-results, ods-select, ods-map, ods-chart, ods-simple-tabs, ods-facets, etc.
  Also activate for: Huwise code editor (HTML+CSS), French open data portals, ODS portal CSS,
  Design Système d'État (DSFR), CSS Grid tables with sticky columns, dataset filtering,
  directory/annuaire layouts (sidebar + cards + map), or any question related to the
  OpenDataSoft/Huwise ecosystem.
version: 1.0.0
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

**⚠️ Piège `ctx-urlsync` + `ctx-parameters` :** Quand `ctx-urlsync="true"` est activé, ODS ignore complètement les `ctx-parameters` définis dans le HTML (warning console : *"There are specific parameters defined, but URL sync is enabled, so the parameters will be ignored"*). Résultat : les filtres `disjunctive`, le `q` de base, etc. ne s'appliquent jamais → filtres dropdown vides, données non filtrées.

**Règle :** n'activer `ctx-urlsync="true"` que si le partage d'URL filtré est explicitement requis. Dans ce cas, initialiser tous les paramètres via des expressions AngularJS `{{ ctx.parameters['...'] = ...; "" }}` plutôt que dans l'attribut HTML.

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

**Pattern — Guard no-results sans flash au chargement :**
Avec `ods-results`, le tableau `lines` est vide (`[]`) pendant le chargement initial — un `ng-if="lines.length == 0"` affiche donc un faux "aucun résultat" à chaque page. Solution : utiliser un KPI compteur et tester `!== null` (pas `!== undefined`).

```html
<!-- Dans le bloc data-hidden (voir section 2.8) -->
<div ods-adv-analysis="totalStats" ods-adv-analysis-context="ctx"
     ods-adv-analysis-select="count(*) as total">
    {{ values.total = (totalStats && totalStats.length > 0) ? totalStats[0].total : null; "" }}
</div>

<!-- Guard : s'affiche UNIQUEMENT quand la donnée est chargée ET vaut 0 -->
<div class="no-results" ng-if="values.total !== null && values.total == 0">
    Aucun résultat ne correspond à votre recherche.
</div>
```

`values.total` part à `null` (jamais `0`) → le bloc reste caché pendant le chargement. Il apparaît uniquement quand l'API confirme 0 résultats.

### 2.6 `ods-map` — Carte synchronisée avec le contexte filtré

```html
<ods-map style="height:520px"
         no-refit="false"
         scroll-wheel-zoom="false"
         display-control="false"
         search-box="false"
         toolbar-fullscreen="true"
         toolbar-geolocation="false"
         basemap="jawg.light"
         location="9,47.89333,-2.80838">
    <ods-map-layer-group>
        <ods-map-layer context="ctx"
                       color="#003e6a"
                       picto="ods-circle"
                       show-marker="true"
                       display="auto"
                       shape-opacity="0.5"
                       point-opacity="1"
                       border-color="#FFFFFF"
                       border-opacity="1"
                       border-size="1"
                       border-pattern="solid"
                       caption="false"
                       size="4"
                       size-min="3"
                       size-max="5"
                       size-function="linear">
        </ods-map-layer>
    </ods-map-layer-group>
</ods-map>
```

**Points clés :**
- `context="ctx"` : la carte est automatiquement synchronisée avec tous les filtres actifs sur `ctx`
- `no-refit="false"` : recadre la vue quand les résultats changent (comportement par défaut souhaitable)
- `location="zoom,lat,lng"` : position initiale (format `"9,47.89333,-2.80838"`)
- `basemap="jawg.light"` : fond de carte clair élégant, disponible sur les portails Huwise
- `display="auto"` : bascule automatiquement entre points et clusters selon le zoom

### 2.7 `ods-select` — Initialisation `ng-init` : piège critique

Le `ods-select` stocke sa valeur dans une variable d'objet (ex. `selected.coll`). Si l'objet parent `selected` n'existe pas encore sur le scope au moment de l'initialisation du widget, AngularJS lève une erreur `Cannot set property 'coll' of undefined`.

**Règle :** toujours initialiser l'objet parent sur l'élément **ancêtre** avant les `ods-select` enfants.

```html
<!-- MAUVAIS — selected n'existe pas quand ods-select s'initialise -->
<div>
    <ods-select ng-init="selected.coll = []" ...></ods-select>
</div>

<!-- BON — selected est créé sur le parent avant les enfants -->
<aside ng-init="selected = {coll: [], prof: []}">
    <ods-select ng-init="selected.coll = []" ...></ods-select>
    <ods-select ng-init="selected.prof = []" ...></ods-select>
</aside>
```

### 2.8 Conteneur data-hidden — Charger des KPIs sans les afficher

Pour calculer des agrégats (totaux, comptes) utilisés comme KPIs ou pour alimenter des filtres, sans rien afficher dans le DOM :

```html
<div style="display:none">

    <!-- Total réactif aux filtres -->
    <div ods-adv-analysis="totalStats"
         ods-adv-analysis-context="ctx"
         ods-adv-analysis-select="count(*) as total">
        {{ values.total = (totalStats && totalStats.length > 0) ? totalStats[0].total : null; "" }}
    </div>

    <!-- COUNT DISTINCT simulé : group by + .length -->
    <div ods-adv-analysis="distinctData"
         ods-adv-analysis-context="ctx"
         ods-adv-analysis-select="count(*)"
         ods-adv-analysis-group-by="mon_champ">
        {{ values.distinctCount = distinctData.length; "" }}
    </div>

    <!-- Options dropdown (contexte fixe, jamais raffiné) -->
    <div ods-adv-analysis="colOptions"
         ods-adv-analysis-context="ctxlist"
         ods-adv-analysis-select="count(*)"
         ods-adv-analysis-group-by="mon_champ"
         ods-adv-analysis-order-by="mon_champ"
         ods-adv-analysis-limit="500">
        {{ values.colOptions = colOptions; "" }}
    </div>

</div>
```

**⚠️ Initialiser `values = {}` sur le scope parent :** `ods-adv-analysis` crée un scope enfant. L'expression `{{ values.colOptions = colOptions; "" }}` remonte la donnée vers le scope parent via mutation de l'objet `values`. Si `values` n'est pas initialisé sur le scope parent, l'assignation échoue silencieusement → filtres vides, KPIs vides.

```html
<div class="data-hidden" ng-init="values = {}">
    <div ods-adv-analysis="colOptions" ...>
        {{ values.colOptions = colOptions; "" }}
    </div>
</div>
```

`ng-init="values = {}"` doit être sur un ancêtre direct de `ods-dataset-context`, **avant** les ods-adv-analysis.

**`value-modifier` : préférer la string simple**
`value-modifier="mon_champ"` (string) → `selected` sera un tableau de valeurs brutes `['Auray', 'Vannes']`, directement utilisable en refine :
```html
{{ ctx.parameters['refine.mon_champ'] = selected.valeurs; "" }}
```
Éviter `value-modifier="{'mon_champ': mon_champ}"` + `| toObject | keys` — plus complexe, inutile pour un refine standard.

**⚠️ NE PAS utiliser `display:none` :** Huwise/ODS ne compile pas les directives AngularJS à l'intérieur d'un élément `display:none`. Les `ods-adv-analysis` ne s'exécutent pas et les variables restent vides (filtres vides, KPIs à 0).

Masquer avec un positionnement hors-écran à la place :
```css
.data-hidden {
    position: absolute;
    left: -9999px;
    top: 0;
    width: 1px;
    height: 1px;
    overflow: hidden;
    pointer-events: none;
}
```

**COUNT DISTINCT :** ODS ne supporte pas `count(distinct champ)` dans `ods-adv-analysis`. Contournement : grouper par le champ et lire `.length` du tableau résultant.

### 2.9 Recherche textuelle personnalisée (sans `ods-text-search`)

`ods-text-search` **remplace entièrement** le paramètre `q` du contexte. Si un filtre de base existe (ex. `q:'#null(date_fin)'`), il sera écrasé.

Solution : `<input>` HTML standard avec `ng-model` + concaténation manuelle du `q`.

```html
<!-- Sidebar ou zone de filtres -->
<div ng-init="search = {query: ''}">
    <input type="text"
           ng-model="search.query"
           ng-model-options="{debounce: 300}"
           placeholder="Rechercher...">
    <button ng-if="search.query" ng-click="search.query = ''">✕</button>
</div>

<!-- Expression de mise à jour du paramètre q -->
{{ ctx.parameters['q'] = search.query
    ? '#null(date_fin_mandat) AND ' + search.query
    : '#null(date_fin_mandat)'; "" }}
```

**Reset complet des filtres :**
```html
<button ng-click="selected.coll = []; selected.prof = []; search.query = '';
                  ctx.parameters['q'] = '#null(date_fin_mandat)';
                  ctx.parameters['refine.mon_champ'] = []">
    Effacer les filtres
</button>
```

**Notes :**
- `ng-model-options="{debounce: 300}"` évite une requête API à chaque frappe
- Le filtre de base (`#null(date_fin_mandat)`) doit figurer dans les deux branches du ternaire et dans le reset

### 2.10 `ods-simple-tabs` — Navigation par onglets

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

## 4. Layout annuaire — Sidebar + Grille de cartes + Carte

Pattern complet pour un annuaire filtrable avec carte synchronisée (testé sur portail Huwise, projet Morbihan Energies).

### Structure HTML

```html
<ods-dataset-context context="ctx, ctxlist"
    ctx-dataset="mon-dataset"
    ctx-parameters="{'disjunctive.champ1':true,'disjunctive.champ2':true,'q':'#null(date_fin)'}"
    ctx-urlsync="true"
    ctxlist-dataset="mon-dataset"
    ctxlist-parameters="{'q':'#null(date_fin)'}">

    <!-- Bloc data-hidden : KPIs + options dropdown -->
    <div style="display:none">...</div>

    <!-- Header avec KPIs -->
    <header class="me-header">...</header>

    <!-- Layout 3 colonnes -->
    <div class="me-layout">

        <!-- Sidebar gauche : recherche + filtres (sticky) -->
        <aside class="me-sidebar" ng-init="selected = {champ1: [], champ2: []}">
            <!-- Recherche custom -->
            <!-- ods-select filtres -->
            <!-- Footer sidebar : compteur + reset -->
        </aside>

        <!-- Zone centrale : grille de cartes -->
        <main class="me-cards-area">
            <div ods-results="items" ods-results-context="ctx" ods-results-max="200">
                <!-- Guard no-results -->
                <!-- ng-repeat cartes -->
            </div>
        </main>

        <!-- Carte ODS droite (sticky) -->
        <aside class="me-map-area">
            <div class="me-map-sticky">
                <ods-map ...>...</ods-map>
            </div>
        </aside>

    </div>
</ods-dataset-context>
```

### CSS de base (layout)

```css
.me-layout {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    gap: 20px;
    max-width: 1440px;
    margin: 0 auto 50px;
    padding: 0 20px;
}

/* Sidebar sticky */
.me-sidebar {
    flex: 0 0 280px;
    position: sticky;
    top: 20px;
    max-height: calc(100vh - 40px);
    overflow-y: auto;
}

/* Zone cartes */
.me-cards-area { flex: 1 1 0; min-width: 0; }

/* Grille responsive */
.me-cards-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
    gap: 16px;
}

/* Carte ODS sticky */
.me-map-area { flex: 0 0 380px; }
.me-map-sticky { position: sticky; top: 20px; }

/* Responsive : empiler sur mobile */
@media (max-width: 1024px) {
    .me-layout { flex-direction: column; }
    .me-sidebar, .me-map-area { flex: none; width: 100%; position: static; }
}
```

**Notes :**
- Le header avec `clip-path` diagonal doit avoir un `padding-bottom` suffisant pour masquer le blanc derrière la diagonale
- `margin-top: -44px` sur `.me-layout` permet au contenu de "monter" sous la pointe du clip-path
- `ods-results-max="200"` : limiter pour les performances (l'utilisateur filtre pour réduire)

---

## 5. Patterns AngularJS récurrents

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

## 6. CSS — Bonnes pratiques pour portails ODS

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

---

## 7. Sortie attendue

Quand on génère du code pour Huwise, fournir :
1. **Un bloc HTML** à coller dans le panneau HTML du code éditeur
2. **Un bloc CSS** à coller dans le panneau CSS du code éditeur
3. Les deux doivent être indépendants et complets

---

## 8. Checklist avant livraison

**Contextes & données :**
- [ ] Tous les contextes nécessaires sont déclarés dans le `ods-dataset-context`
- [ ] Les contextes pour listes d'options (`ctxlist`) sont séparés des contextes filtrés (`ctx`)
- [ ] Le `disjunctive` est activé sur les champs multi-refine
- [ ] Le filtre de base (`q`) est présent dans les paramètres initiaux ET dans toutes les expressions de mise à jour

**AngularJS :**
- [ ] Les `ng-repeat` utilisent `track by $index`
- [ ] L'objet parent est initialisé avant les propriétés enfants (`ng-init="selected = {a:[], b:[]}"` sur l'ancêtre)
- [ ] La recherche textuelle utilise un `<input>` custom + concaténation `q` (pas `ods-text-search`) si un filtre de base existe
- [ ] Le guard no-results utilise `values.total !== null && values.total == 0` (pas `lines.length == 0`)

**CSS :**
- [ ] Les colonnes sticky ont un `background` explicite
- [ ] Le `z-index` du header est supérieur à celui des lignes
- [ ] Le CSS utilise les variables du portail quand elles existent
- [ ] Les nombres sont formatés avec le pipe `number` et des parenthèses
- [ ] Le layout responsive est prévu (`flex-direction: column` sous 1024px)

**Carte ODS :**
- [ ] La carte partage le même `context="ctx"` que les filtres pour être synchronisée
- [ ] `no-refit="false"` si on veut que la vue se recadre après filtrage
