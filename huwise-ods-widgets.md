---
name: huwise-ods-widgets
description: >
  This skill should be used when developing pages or dashboards on the Huwise (OpenDataSoft) platform
  with ODS widgets and AngularJS directives — including ods-dataset-context, ods-adv-analysis,
  ods-results, ods-select, ods-map, ods-chart, ods-simple-tabs, ods-facets, etc.
  Also activate for: Huwise code editor (HTML+CSS), French open data portals, ODS portal CSS,
  Design Système d'État (DSFR), CSS Grid tables with sticky columns, dataset filtering,
  directory/annuaire layouts (sidebar + cards + map), debugging ODSQL query errors
  (Unknown field, StatAggregation type errors, blank pages caused by a master ng-if),
  field typing/renaming issues between datasets, or any question related to the
  OpenDataSoft/Huwise ecosystem.
version: 1.2.0
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
- Et notamment ODS Map (doc directive) : https://help.opendatasoft.com/widgets/#/api/ods-widgets.directive:odsMap

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

**Pattern — Cibler des valeurs précises (et non le « top N ») :**
`order-by="-count"` + `limit="N"` renvoie mécaniquement les N groupes les plus nombreux — impossible de garantir que telle ou telle valeur en fasse partie. Pour afficher des valeurs **choisies** quel que soit leur rang, les désigner explicitement avec `in (...)` (le tri `-count` ordonne alors seulement ces valeurs) :
```html
<div ods-adv-analysis="lignes"
     ods-adv-analysis-context="ctx"
     ods-adv-analysis-where="mon_champ in ('Valeur A','Valeur B','Valeur C')"
     ods-adv-analysis-group-by="mon_champ as type"
     ods-adv-analysis-order-by="-count"
     ods-adv-analysis-select="count(*) as count">
```

**⚠️ Piège — Apostrophe (ou guillemet) dans une valeur du `where` :**
Une **apostrophe droite** `'` (U+0027) présente DANS une valeur casse un littéral ODSQL délimité par `'` → réponse **400**. La parade fiable : délimiter le littéral par des **guillemets doubles**, encodés `&quot;` dans l'attribut HTML (l'attribut est déjà entre `"`). Doubler l'apostrophe (`''`) échoue ; `like 'x%y'` ne fait PAS de wildcard non plus.
```html
<!-- valeur "Equipement d'athlétisme" (apostrophe droite) -->
ods-adv-analysis-where="mon_champ in (&quot;Bassin de natation&quot;,&quot;Equipement d'athlétisme&quot;)"
```
> Toujours **tester le `where` via l'API `records`** avant de livrer : les erreurs ODSQL d'`ods-adv-analysis` sont loguées silencieusement par gulp (page vide/ligne manquante sans message). Ex. : `.../api/explore/v2.1/catalog/datasets/<id>/records?where=...&group_by=...`.

**Pattern — Ajouter des lignes issues d'un SECOND champ de regroupement (mélange de granularités) :**
Pour afficher dans un même tableau des lignes agrégées par un champ fin (ex. `equipement_type`) ET des lignes agrégées par un champ plus regroupé (ex. `equipement_famille`), **imbriquer un second jeu d'`ods-adv-analysis`** groupé par l'autre champ (réutiliser le même alias, ex. `... as type`, pour partager le markup de cellule), puis ajouter une **2ᵉ boucle `ng-repeat`** dans le même `<tbody>`. Les scopes enfants créés par `ods-adv-analysis` rendent les deux jeux de variables accessibles à la boucle interne.
```html
<div ods-adv-analysis="lignes_type"  ... group-by="equipement_type as type"    order-by="-count" limit="5">
  <div ods-adv-analysis="lignes_fam" ... group-by="equipement_famille as type" order-by="-count" limit="2">
    <table><tbody>
      <tr ng-repeat="r in lignes_type"> ... {{r.type}} ... </tr>
      <tr ng-repeat="r in lignes_fam">  ... {{r.type}} ... </tr>  <!-- lignes ajoutées -->
    </tbody></table>
  </div>
</div>
```
⚠️ Vérifier l'équilibre des `<div>` : ces wrappers `ods-adv-analysis` sont souvent laissés non fermés dans le code existant (ils englobent tout ce qui suit). Si on les ferme, garder la profondeur d'imbrication identique pour ne pas décaler la mise en page en aval.

**Pattern — Colonnes « X déclarées (Y %) » (moyenne sur champ non nul + comptage) :**
Pour une moyenne sur un champ (ex. année) accompagnée du nombre de lignes qui l'ont réellement renseigné : deux agrégations parallèles — une avec `where "... and champ is not null"` (le nombre « déclaré »), une sans (le total du groupe) — puis calcul du % côté template. On récupère la valeur du groupe courant par `(total | filter:{type:r.type})[0].count`.
```html
{{ (total_declares | filter:{type:r.type})[0].count | number:0 }} déclarées
({{ ((total_declares | filter:{type:r.type})[0].count / r.count) * 100 | number:1 }}%)
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

**Pagination — la directive s'appelle `ods-pagination-block` (PAS `ods-pagination`) :**
`ods-pagination` n'existe pas et ne fait rien au copier-coller. Le bon widget est [`odsPaginationBlock`](https://help.opendatasoft.com/widgets/#/api/ods-widgets.directive:odsPaginationBlock) → balise `ods-pagination-block`.

```html
<div ods-results="lines" ods-results-context="ctx" ods-results-max="12">
    <div ng-repeat="line in lines track by $index">{{ line.fields.nom }}</div>
</div>
<ods-pagination-block context="ctx" per-page="12" nofollow="true"></ods-pagination-block>
```

- `per-page` doit correspondre à `ods-results-max` ; le widget met à jour `start` sur le contexte, et `ods-results` recharge la bonne tranche.
- `nofollow="true"` pour ne pas faire suivre les liens de pagination par les crawlers.

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

#### `ods-map` + `refine-on-click` — sélection par clic sur un point (point → panneau de détails)

Pattern « clic sur un point de la carte → refine d'un **autre** contexte → panneau de détails à côté » (cf. codelibrary « refine on the side »). Deux contextes sur le même dataset : `map` (tous les points, couche cliquable) et `sel` (raffiné au clic).

```html
<ods-dataset-context context="map,sel" map-dataset="etabs" sel-dataset="etabs">
  <ods-map basemap="jawg.light" location="8,49.1,0.1" no-refit="true" style="height:560px">
    <ods-map-layer context="map"
                   display="raw"                              <!-- OBLIGATOIRE pour refine-on-click -->
                   show-marker="true"
                   refine-on-click-context="sel"
                   refine-on-click-sel-map-field="code_etab"       <!-- champ lu sur le point cliqué -->
                   refine-on-click-sel-context-field="code_etab"   <!-- champ raffiné dans le contexte sel -->
                   refine-on-click-sel-replace-refine="true">      <!-- chaque clic remplace la sélection -->
    </ods-map-layer>
    <!-- point sélectionné mis en évidence -->
    <ods-map-layer context="sel" show-if="sel.parameters['refine.code_etab']" color="#c9191e" picto-size="7"></ods-map-layer>
  </ods-map>
  <div ng-if="sel.parameters['refine.code_etab']">
    <ods-result-enumerator context="sel" max="1"> … détails du point sélectionné … </ods-result-enumerator>
  </div>
</ods-dataset-context>
```

**PIÈGES bloquants (débuggés sur portail Huwise, `ods-widgets latest-v2`, via navigateur) :**
- **Une `ods-map` SANS attribut `location` ne charge PAS ses points** sur ces portails : aucune requête `boundingbox`/`geocluster` n'est émise, la carte reste figée (spinner infini ou vue par défaut sur Paris). → **TOUJOURS** fournir `location="zoom,lat,lng"` (l'auto-fit sans `location` ne suffit pas).
- **`refine-on-click` FIGE toute la carte (0 tuile, spinner infini, aucune erreur console) SAUF si la couche est en `display="raw"` (ou `"aggregation"`)** — en mode marqueur/cluster par défaut, ça ne marche pas. Confirmé dans le source ods-widgets (« If a layer is displayed as `raw` or `aggregation`, it can be configured so that a click on an item triggers a refine on another context »).
- `display="raw"` télécharge les records individuellement (endpoint `download`, `rows≤1000`, `geo_simplify`) → **laisser quelques secondes** avant l'apparition des marqueurs ; attention aux points dupliqués superposés (un même identifiant sur plusieurs lignes = plusieurs marqueurs au même endroit, le clic reste correct).
- Attributs (valables sur `ods-map-layer`, doc source) : `refine-on-click-context="ctx"` (ou `"[ctx1, ctx2]"`), `refine-on-click-<ctx>-map-field`, `refine-on-click-<ctx>-context-field`, `refine-on-click-<ctx>-replace-refine="true"`. Le contexte raffiné peut être **différent** de celui de la couche cliquée.
- Masquer le panneau de détails tant qu'aucun clic n'a eu lieu : `ng-if="sel.parameters['refine.<champ>']"` (falsy tant que non raffiné).

#### `ods-map` — Cartes choroplèthes (choropleth par département)

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

#### Légende custom inlined pour `ods-map`

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

### 2.11 Onglets-comme-filtre — barre de pills custom pilotant un refine

**Quand l'utiliser :** quand on veut une navigation « par onglets » qui **filtre un dataset partagé** (une seule carte, un seul graphique, une seule liste qui réagissent à l'onglet actif).

**⚠️ Ne PAS utiliser `ods-simple-tabs` pour ça.** `ods-simple-tabs` sert uniquement à afficher/masquer du **contenu** : chaque onglet contient son propre DOM. Pour filtrer un contexte partagé, il faudrait soit dupliquer toutes les visualisations dans chaque onglet (N cartes, N graphiques → ingérable), soit s'appuyer sur le `ng-init` du contenu d'onglet (fragile, dépend de `keep-content`). 

**Solution robuste : une barre de boutons `ng-click` qui pilote une variable de modèle, synchronisée vers le refine.** Le rendu visuel est identique à des onglets/pills.

```html
<!-- Modèle initialisé sur un ancêtre -->
<div class="page" ng-init="memorial = {conflit:null}">
  <ods-dataset-context context="ctx" ctx-dataset="..." ctx-domain="...">

    <!-- Synchronisation refine (caché, toujours rendu → s'exécute à chaque digest) -->
    <span class="data-hidden">{{ ctx.parameters['refine.conflit'] = memorial.conflit ? memorial.conflit : undefined; '' }}</span>

    <!-- Barre d'onglets -->
    <nav class="tabs">
      <button type="button" ng-class="{'is-active': !memorial.conflit}"
              ng-click="memorial.conflit = null">Tous</button>
      <button type="button" ng-class="{'is-active': memorial.conflit == 'Première Guerre mondiale'}"
              ng-click="memorial.conflit = 'Première Guerre mondiale'">1914-1918</button>
      <!-- valeur avec apostrophe → échapper avec \' -->
      <button type="button" ng-class="{'is-active': memorial.conflit == 'Guerre d\'Indochine'}"
              ng-click="memorial.conflit = 'Guerre d\'Indochine'">Indochine</button>
    </nav>

    <!-- Visualisations UNIQUES bound à ctx → réagissent à l'onglet -->
    <ods-map>...</ods-map>
    <div ods-results="lines" ods-results-context="ctx">...</div>
  </ods-dataset-context>
</div>
```

**Points clés :**
- **Effacer un refine** : assigner `undefined` (pas `''` ni `null` → renverraient 0 résultat). ods-widgets omet les paramètres `undefined` à la sérialisation. L'onglet « Tous » remet donc `memorial.conflit = null` → l'expression assigne `undefined`.
- **Idempotence** : l'expression `{{ ctx.parameters['refine.x'] = ... ; '' }}` réassigne la même valeur à chaque digest → pas d'infinite digest (valeur stable). Même principe que le `q` custom (section 2.9).
- **Apostrophes dans les valeurs** (`Guerre d'Indochine`, `Autres théâtres d'opérations`) : dans un attribut HTML en guillemets doubles, échapper l'apostrophe interne avec `\'` — `ng-click="memorial.conflit = 'Guerre d\'Indochine'"`. Ne **pas** utiliser `&#39;` (le parser HTML le décode en `'` avant Angular et casse la chaîne).
- La valeur du refine doit correspondre **exactement** à la valeur du dataset (vérifier via l'endpoint `/facets`).

### 2.12 KPIs réactifs au filtre — `ods-adv-analysis` avec `where`

Pour des KPIs (total, sous-totaux) qui se **recalculent selon le filtre actif**, utiliser plusieurs `ods-adv-analysis` sur le **contexte filtré `ctx`**, avec une clause `where` pour les sous-ensembles. Comme ils pointent sur `ctx`, ils héritent automatiquement du `refine` posé par les onglets.

```html
<span class="data-hidden" ods-adv-analysis="kpiTotal" ods-adv-analysis-context="ctx"
      ods-adv-analysis-select="count(*) as nb">{{ values.total = kpiTotal[0].nb; '' }}</span>

<span class="data-hidden" ods-adv-analysis="kpiMer" ods-adv-analysis-context="ctx"
      ods-adv-analysis-select="count(*) as nb"
      ods-adv-analysis-where="code_pays_deces LIKE 'MER'">{{ values.mer = kpiMer[0].nb; '' }}</span>

<span class="data-hidden" ods-adv-analysis="kpiMil" ods-adv-analysis-context="ctx"
      ods-adv-analysis-select="count(*) as nb"
      ods-adv-analysis-where="categorie LIKE 'Militaire%'">{{ values.mil = kpiMil[0].nb; '' }}</span>
```

- `where` accepte l'ODSQL : `LIKE 'valeur'` (égalité sur valeur exacte), `LIKE 'prefixe%'` (préfixe avec joker `%`).
- Initialiser `values = {}` sur un ancêtre (cf. 2.8) sinon les assignations échouent silencieusement.
- Pour afficher pendant le chargement : `{{ (values.total | number:0) || '…' }}`.

### 2.13 Barres de proportion en CSS pur (sans `ods-chart`)

Pour un top-N lisible et léger (ex. principaux lieux), rendre des barres à partir d'un `ods-adv-analysis` groupé, en calculant la largeur par rapport au max :

```html
<span class="data-hidden" ods-adv-analysis="paysData" ods-adv-analysis-context="ctx"
      ods-adv-analysis-select="count(*) as nb"
      ods-adv-analysis-group-by="code_pays_deces"
      ods-adv-analysis-order-by="nb desc"
      ods-adv-analysis-limit="8">{{ values.pays = paysData; values.paysMax = (paysData[0].nb || 1); '' }}</span>

<li ng-repeat="p in values.pays track by $index" ng-if="p.code_pays_deces">
  <span>{{ paysLabels[p.code_pays_deces] || p.code_pays_deces }}</span>
  <span>{{ p.nb | number:0 }}</span>
  <div class="bar__track"><div class="bar__fill" style="width: {{ (p.nb / values.paysMax) * 100 }}%"></div></div>
</li>
```

- Le champ de regroupement devient une **propriété de l'objet résultat** (`p.code_pays_deces`), à côté des alias du `select` (`p.nb`).
- Un groupe `null` apparaît si le champ est vide → filtrer avec `ng-if="p.champ"`.
- Mapper des codes vers des libellés lisibles via un objet `ng-init` (`paysLabels = {'FRA':'France', ...}`) + accès `paysLabels[code] || code`.

### 2.14 Carte en clusters de points (`display="clusters"`)

Pour une carte « où sont les événements » à partir de points (`geo_point_2d`), utiliser `display="clusters"` : agrège les points proches en pastilles numérotées, idéal pour des milliers de records.

```html
<ods-map no-refit="true" scroll-wheel-zoom="false" basemap="jawg.light" location="5,47.5,3">
  <ods-map-layer-group>
    <ods-map-layer context="ctx" display="clusters" color="#da8e2b" picto="circle" show-marker="true"></ods-map-layer>
  </ods-map-layer-group>
</ods-map>
```

- Synchronisée avec `ctx` → réagit aux onglets/filtres.
- `no-refit="true"` fige le cadrage choisi (sinon des points lointains — ex. décès à l'étranger — feraient dézoomer la carte au niveau monde et écraseraient le cluster local).
- Rappel : zoom **entier** dans `location` (cf. section 2.6).

### 2.15 Formater une date ISO sans locale

Les champs date ODS sont des chaînes ISO (`"1916-02-25"`). Le filtre AngularJS natif `date` les parse directement :

```html
{{ p.fields.date_deces | date:'dd/MM/yyyy' }}   <!-- → 25/02/1916 -->
```

Préférer un format **numérique** (`dd/MM/yyyy`) : les formats à mois littéral (`MMMM`) dépendent de la locale Angular (souvent `en`) et afficheraient « February ».

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

## 6. Debug ODSQL — erreurs de requête et typage des champs

Diagnostic des erreurs ODSQL qui s'affichent dans la console du navigateur. Elles proviennent des requêtes envoyées à l'API par `ods-adv-analysis`, `ods-chart-query` et tout `select`/`where`/agrégation. Une page peut avoir **plusieurs erreurs identiques + une différente** : c'est souvent la différente qui casse tout.

### 6.1 La page entière ne s'affiche pas (même pas le CSS)

Quand **rien** ne se rend — ni les données, ni la mise en page — la cause est presque toujours un **`ng-if` maître** qui enveloppe tout le corps de la page et dépend d'une variable alimentée par une requête en échec.

```html
<!-- Une analyse calcule la dernière année -->
<div ods-adv-analysis="yearData" ods-adv-analysis-context="ctxyear"
     ods-adv-analysis-select="max(annee) as derniere">
    {{ variables.derniere_annee = yearData[0].derniere; "" }}
</div>
...
<!-- ...et tout le corps est conditionné par cette variable -->
<div ng-if="variables.derniere_annee">
    <!-- TOUTE la page -->
</div>
```

Si `max(annee)` lève une erreur ODSQL, `variables.derniere_annee` reste `undefined` → le `ng-if` masque tout → page blanche, CSS compris.

**Méthode de diagnostic :**
1. Ouvrir la console et lire **chaque** message ODSQL, pas seulement le premier.
2. Repérer la variable qui pilote le `ng-if` maître, remonter à l'analyse qui l'alimente — c'est l'« interrupteur » de la page.
3. Vérifier le champ incriminé **par curl sur l'API** avant de toucher au code (`gulp` log les erreurs API silencieusement).

### 6.2 « Unknown field: X »

Le champ référencé dans un `select`/`group-by`/`where` n'existe pas dans **ce** dataset.

Causes fréquentes :
- **Colonne renommée en preprod** (régression) — ex. `deces_nb` devenu `deces_tot_nb` dans la version preprod du dataset.
- **Même nom logique, datasets différents** — un champ peut exister dans certains datasets (ou copies) et pas dans d'autres. Un même nom de variable peut donc être **valide sur un contexte et invalide sur un autre**.

⚠️ **Ne jamais faire un rechercher-remplacer global.** Lister les contextes par dataset, identifier ceux qui pointent le dataset fautif, et corriger **uniquement** ceux-là. Les contextes qui visent un autre dataset où le champ existe ne doivent pas être touchés.

### 6.3 « StatAggregation only supports numeric or date expression »

Une fonction d'agrégation (`max`, `min`, `sum`, `avg`…) est appliquée à un champ **typé string**. Ex. `max(annee)` échoue si `annee` est typé texte dans le dataset.

Deux solutions :
- **Retyper le champ** (numeric ou date) dans le back-office du dataset puis republier — corrige sans toucher au code (souvent le plus propre).
- **Caster dans la requête** si le retypage est impossible.

⚠️ **Effet de bord du retypage en date sur les filtres.** Un `where`/`q` qui compare le champ à un entier fonctionne sur un champ numérique mais peut casser une fois le champ retypé en **date** :

```
-- Casse potentiellement sur un champ date :
annee >= 2021 AND annee <= 2024

-- Adapter :
year(annee) >= 2021 AND year(annee) <= 2024
-- ou
annee >= date'2021-01-01' AND annee <= date'2024-12-31'
```

Après un retypage de dataset, retester systématiquement les filtres `q`/`where` qui référencent le champ.

### 6.4 Règle d'or — vérifier le dataset avant de corriger le code

Les erreurs ODSQL traduisent un **écart entre le code et le schéma réel du dataset** (champ renommé, retypé, supprimé). Avant toute correction de code, confirmer le **nom exact** et le **type exact** du champ par curl sur l'API du dataset. Corriger le code à l'aveugle fait courir le risque de « réparer » un contexte sain ou de masquer la vraie régression côté dataset.

## 7. CSS — Bonnes pratiques pour portails ODS

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

### Conformité DSFR (Design Système de l'État)
Pour les portails gouvernementaux français :
- Police Marianne obligatoire
- Classes préfixées `fr-` (fr-container, fr-grid-row, fr-card, etc.)
- Tokens couleur officiels : bleu France `#000091`, etc.
- Système d'espacement basé sur multiples de 8px

### DSFR sur portails Huwise — pièges « thème partiel »

Le `theme.scss` d'un portail Huwise est souvent un **build DSFR partiel** : certains composants n'ont **aucun style** (ni JS). Toujours vérifier avant de les utiliser :
```bash
grep -c "fr-notice" theme.scss   # 0 → composant absent, à styler soi-même
grep -c "fr-alert"  theme.scss
```
Les **tokens couleur** (`--blue-france-*`, `--background-contrast-info`, `--text-default-info`, `--grey-*`) sont généralement présents même quand les composants manquent.

**Bandeau d'information → `fr-notice`, pas `fr-alert`.** L'icône de `fr-alert` est injectée via `::before { mask-image: url(...) }` pointant vers un asset DSFR souvent non déployé → alerte vide/cassée. Utiliser la structure `fr-notice` avec icône explicite + CSS embarqué dans le `.scss` de la page :
```html
<div class="fr-notice fr-notice--info">
  <div class="fr-notice__body">
    <p>
      <span class="fr-notice__title"><img src="/assets/theme_image/fr--info-fill.svg" alt="" aria-hidden="true" class="notice-icon"></span>
      <span class="fr-notice__desc">Votre message d'information.</span>
    </p>
  </div>
</div>
```
```css
.fr-notice { background-color: var(--background-contrast-info); color: var(--text-default-info); border-radius: .25rem; }
.fr-notice .fr-notice__body { padding: 1rem 1.5rem; }
.fr-notice .fr-notice__body p { margin: 0; display: flex; align-items: flex-start; gap: .5rem; }
.notice-icon { width: 1.25rem; height: 1.25rem; display: inline-block; }
/* recolorer un SVG noir en bleu info via filter, ou utiliser une icône Font Awesome <i class="fa fa-info-circle"> (FA est chargé) */
```

**Boutons DSFR tronqués.** Un `.fr-btn` portant une classe `fr-icon-*` **sans** modificateur `fr-btn--icon-*` est clampé par le DSFR à `max-width:2.5rem; overflow:hidden; white-space:nowrap` (style « bouton icône-seule ») → le label disparaît. Pour un **bouton-lien externe**, ne pas mettre `fr-icon-external-link-line` : utiliser `class="fr-btn" target="_blank"` → le DSFR ajoute l'icône lien-externe via `::after` ET applique `max-width:100%; overflow:initial`. Garde-fou : `.fr-btns-group .fr-btn { white-space:normal; max-width:100%; height:auto; }`.

**Accordéons sans la JS DSFR.** La JS DSFR n'est pas chargée sur Huwise → câbler le repli en **AngularJS** (état primitif sur le scope, pas de `display:none`). Pattern, avec un seul accordéon ouvert à la fois :
```html
<div class="fr-accordions-group" ng-init="acc1='collapsed'; acc2='collapsed';">
  <div class="fr-accordion__section">
    <button class="fr-accordion__btn fr-accordion__btn-{{acc1}}"
            ng-click="acc1=(acc1==='collapsed')?'extended':'collapsed'; acc2='collapsed';">
      <p class="fr-text-accordion">Titre</p>
      <i style="margin-left:auto" class="fa {{acc1==='collapsed'?'fa-angle-down':'fa-angle-up'}}" aria-hidden="true"></i>
    </button>
    <div class="fr-accordion__container fr-accordion__container-{{acc1}}"> … </div>
  </div>
</div>
```
```css
.fr-accordion__btn { width:100%; display:flex; justify-content:space-between; align-items:center; }
.fr-accordion__btn-collapsed { background:var(--grey-1000); }
.fr-accordion__btn-extended  { background:var(--blue-france-925-125); }
.fr-accordion__container-collapsed { max-height:0; overflow:hidden; padding:0 2em; }
.fr-accordion__container-extended  { max-height:100%; padding:2rem 2rem 3rem; }
```
Veiller à ce qu'aucune directive créant un scope (ex. `ods-simple-tab`) ne s'intercale entre le `ng-init` et les `ng-click`/`{{ }}`, sinon l'écriture d'un primitif crée une copie masquée (shadow) qui casse le binding.

**Rappel API :** les attributs HTML `on*` (`onclick`…) sont **bloqués par l'API ODS** au `gulp put` → pas de bouton de fermeture `onclick` sur une alerte ; gérer toute interactivité en directives AngularJS (`ng-click`).

---

## 8. Sortie attendue

Quand on génère du code pour Huwise, fournir :
1. **Un bloc HTML** à coller dans le panneau HTML du code éditeur
2. **Un bloc CSS** à coller dans le panneau CSS du code éditeur
3. Les deux doivent être indépendants et complets

---

## 9. Checklist avant livraison

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
- [ ] Navigation « par onglets » qui filtre un dataset → barre de boutons `ng-click` pilotant un refine (PAS `ods-simple-tabs`, cf. 2.11)
- [ ] Effacer un refine via expression → assigner `undefined` (jamais `''` ni `null`)
- [ ] Apostrophes dans les valeurs `ng-click`/`ng-class` échappées avec `\'` (pas `&#39;`)
- [ ] Dates ISO formatées avec `| date:'dd/MM/yyyy'` (format numérique, indépendant de la locale)

**CSS :**
- [ ] Les colonnes sticky ont un `background` explicite
- [ ] Le `z-index` du header est supérieur à celui des lignes
- [ ] Le CSS utilise les variables du portail quand elles existent
- [ ] Les nombres sont formatés avec le pipe `number` et des parenthèses
- [ ] Le layout responsive est prévu (`flex-direction: column` sous 1024px)

**Carte ODS :**
- [ ] La carte partage le même `context="ctx"` que les filtres pour être synchronisée
- [ ] `no-refit="false"` si on veut que la vue se recadre après filtrage
- [ ] Chaque message ODSQL de la console est lu (pas seulement le premier) — identifier l'erreur « différente » qui éteint la page
- [ ] Si page blanche : remonter le `ng-if` maître jusqu'à l'analyse qui l'alimente
- [ ] Nom et type des champs incriminés vérifiés **par curl** sur l'API avant correction
- [ ] « Unknown field » corrigé **sélectivement** par contexte/dataset, jamais en rechercher-remplacer global
- [ ] Aucune agrégation (`max`/`min`/`sum`/`avg`) sur un champ typé string
- [ ] Après retypage d'un champ en date : filtres `q`/`where` comparant à un entier retestés (`year(...)` ou `date'...'`)
