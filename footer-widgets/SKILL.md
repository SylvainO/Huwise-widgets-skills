---
name: footer-widgets
description: "Configuration et personnalisation du footer (pied de page) sur la plateforme Huwise (ex-OpenDataSoft). Utiliser cette skill dès que le contexte implique : footer de portail Huwise/ODS, pied de page, bas de page, placeholders footer (##legal##, ##manage-cookies##, ##huwise-logo##, ##language##, ##accessibility##), configuration du footer dans le back-office (Portail, Style, Pied de page), feuille de style footer, classes .ods-front-footer__, classes DSFR .fr-footer__, liens légaux, gestion des cookies, licence ouverte, logos partenaires dans le footer, réseaux sociaux footer, ou toute question liée au pied de page d'un portail OpenDataSoft/Huwise. Même environnement technique que huwise-ods-widgets (AngularJS, CSS, portails données publiques françaises, DSFR)."
---

# Footer Huwise / ODS — Skill de référence

## 1. Contexte technique

### Plateforme
- **Huwise** (anciennement OpenDataSoft) — plateforme de données ouvertes
- Le footer se configure au **niveau du domaine Huwise** (back-office), pas dans le code éditeur des pages
- Stack : **HTML/CSS + AngularJS** (directives Angular standard utilisables : `ng-if`, `ng-class`, `ng-show`, `ng-hide`)
- **JavaScript interdit** dans le footer pour raisons de sécurité (comme dans le header)

### Où ça se configure dans le back-office
- **HTML du footer** : Portail > Style > onglet **Pied de page**
- **CSS du footer** : Portail > Style > onglet **Feuille de Style**
- Le footer est **commun à toutes les pages** du portail
- Le thème est **versionné** : chaque modification crée un brouillon à activer via "Rendre cette version active"

### Documentation de référence
- Doc thèmes portail : https://userguide.huwise.com/fr/articles/2042818
- Doc widgets ODS : https://help.opendatasoft.com/widgets/#/introduction/
- GitHub widgets : https://github.com/opendatasoft/ods-widgets
- DSFR footer : https://www.systeme-de-design.gouv.fr/composants-et-modeles/composants/pied-de-page

---

## 2. Footer par défaut Huwise

Voici le HTML par défaut d'un nouveau portail Huwise (avant toute customisation) :

```html
<div class="ods-front-footer">
    ##huwise-logo## ##legal## ##language## ##manage-cookies## ##accessibility##
</div>
```

Le footer par défaut est rendu dans un `<footer>` wrappé par la plateforme avec `ng-controller="FooterController"`. Ce controller AngularJS est injecté automatiquement — il n'est **pas** à déclarer manuellement.

### Placeholders disponibles dans le footer

| Placeholder | Description |
|---|---|
| `##huwise-logo##` | Logo Huwise (anciennement `##ods-logo##`) |
| `##legal##` | Lien vers les Conditions Générales d'utilisation |
| `##manage-cookies##` | Bouton/lien gestion des cookies (obligatoire si bannière cookies active) |
| `##language##` | Sélecteur de langue du portail |
| `##accessibility##` | Lien vers la déclaration d'accessibilité |

> **Note :** `##manage-cookies##` génère la directive `<ods-manage-cookies-preferences>` avec tous ses attributs de configuration de tracking (matomo, GA, etc.). Elle s'insère nativement et hérite des classes `.ods-manage-cookies-preferences__show-button`.

---

## 3. Classes CSS du footer

### Classes natives ODS (`ods-front-footer__*`)

Convention BEM identique au header : `.ods-block[--blockmodifier][__element][--elementmodifier]`

| Classe | Rôle |
|---|---|
| `.ods-front-footer` | Conteneur racine du footer (appliqué sur `<div>` ou `<footer>`) |
| `.ods-front-footer__back` | Wrapper de fond (pour couleur/border-top pleine largeur) |
| `.ods-front-footer__container` | Conteneur centré avec max-width |
| `.ods-front-footer__body` | Zone principale (logo + liens externes) — flex row |
| `.ods-front-footer__logo` | Logo image dans le footer |
| `.ods-front-footer__external-links` | Groupe de liens externes (ferrés à droite) |
| `.ods-front-footer__link` | Style des liens du footer |
| `.ods-front-footer__line` | Barre de séparation + liste de liens du bas |
| `.ods-front-footer__bottom-list` | Liste `<ul>` des liens légaux |
| `.ods-front-footer__bottom-item` | Item `<li>` de la liste |
| `.ods-front-footer__legal` | Lien légal spécifique |
| `.ods-front-footer__licence` | Texte de licence (ex: Licence Ouverte 2.0) |

### Classes DSFR (`fr-footer__*`)

Pour les portails conformes DSFR (portails gouvernementaux) :

| Classe | Rôle |
|---|---|
| `.fr-footer` | Conteneur racine DSFR du footer |
| `.fr-footer__body` | Zone haute (logo + liens institutionnels) — flex wrap |
| `.fr-footer__brand` | Bloc logo/marque — `flex-basis: 100%` |
| `.fr-footer__brand.fr-enlarge-link` | Logo avec zone de clic élargie |
| `.fr-logo` | Logo RF en texte ("Gouvernement") |
| `.fr-footer__content` | Zone des liens institutionnels |
| `.fr-footer__content-list` | Liste des liens institutionnels (flex wrap) |
| `.fr-footer__content-item` | Item de la liste |
| `.fr-footer__content-link` | Lien institutionnel (bold, 0.875rem) |
| `.fr-footer__bottom` | Barre du bas (liens légaux + copyright) |
| `.fr-footer__bottom-list` | Liste des liens légaux |
| `.fr-footer__bottom-item` | Item avec séparateur vertical automatique (`::before`) |
| `.fr-footer__bottom-link` | Lien légal (0.75rem, `--text-mention-grey`) |
| `.fr-footer__bottom-copy` | Zone copyright |
| `.fr-footer__top` | Section optionnelle au-dessus du body (fond gris, colonnes de liens) |
| `.fr-footer__partners` | Section logos partenaires |

---

## 4. Patterns et bonnes pratiques

### Directives AngularJS utilisables
Le footer partage le contexte AngularJS du portail. On peut utiliser :
- `ng-if` / `ng-show` / `ng-hide` — affichage conditionnel
- `ng-class` — classes dynamiques
- `ng-non-bindable` — sur les éléments dont le contenu ne doit **pas** être interpolé par Angular (ex: `src` d'images avec des `{}`)
- Mais **pas de JavaScript** ni de contrôleurs custom

> **`ng-non-bindable` important** : Ajouter `ng-non-bindable` sur les `<img>` ou tout élément dont l'attribut contient des accolades `{}` qui pourraient être interprétées comme des expressions Angular.

### Workflow de modification
1. Aller dans **Portail > Style**
2. Modifier le HTML dans l'onglet **Pied de page** et/ou le CSS dans **Feuille de Style**
3. Cliquer **Enregistrer**
4. Vérifier avec le bouton **Aperçu**
5. Quand satisfait, cliquer **Rendre cette version active**

### Responsive
- Utiliser `@media (max-width: 991px)` pour adapter le layout en mobile (ex: passer de `flex-row` à `flex-column`)
- Contrairement au header, le footer **n'utilise pas** `ods-responsive-menu` — le responsive est géré uniquement en CSS

### Font Awesome dans le footer
Les icônes Font Awesome sont disponibles (chargées par la plateforme) :
```html
<i class="fa fa-external-link" aria-hidden="true"></i>  <!-- lien externe -->
<i class="fa fa-brands fa-linkedin" aria-hidden="true"></i>  <!-- réseaux sociaux -->
```

---

## 5. Pattern A — Footer ODS natif customisé (ex: Équipements sportifs)

Pattern recommandé quand on veut **garder les classes ODS** avec un design custom.

### HTML back-office
```html
<footer>
    <div class="ods-front-footer__back">
        <div class="ods-front-footer__container">
            <!-- Zone haute : logo + liens externes -->
            <div class="ods-front-footer__body">
                <img ng-non-bindable
                     class="ods-front-footer__logo"
                     src="/assets/theme_image/mon-logo.png"
                     alt="Logo de l'organisation">
                <div class="ods-front-footer__external-links">
                    <a href="https://www.mon-site.fr/" class="ods-front-footer__link" target="_blank">
                        mon-site.fr&nbsp;&nbsp;<i class="fa fa-external-link" aria-hidden="true"></i>
                    </a>
                    <a href="https://data.gouv.fr/" class="ods-front-footer__link" target="_blank">
                        data.gouv.fr&nbsp;&nbsp;<i class="fa fa-external-link" aria-hidden="true"></i>
                    </a>
                </div>
            </div>
            <!-- Zone basse : liens légaux + licence -->
            <div class="ods-front-footer__line">
                <ul class="ods-front-footer__bottom-list">
                    <li class="ods-front-footer__bottom-item">
                        <a href="/pages/accueil/" class="ods-front-footer__link">Accueil</a>
                    </li>
                    <li class="ods-front-footer__bottom-item">
                        <a href="/pages/plan-du-site/" class="ods-front-footer__link">Plan du site</a>
                    </li>
                    <li class="ods-front-footer__bottom-item">
                        <a href="/pages/contact/" class="ods-front-footer__link">Contact</a>
                    </li>
                    <li class="ods-front-footer__bottom-item">##manage-cookies##</li>
                    <li class="ods-front-footer__bottom-item">##legal##</li>
                </ul>
                <p class="ods-front-footer__licence">
                    Sauf indication contraire, tout le contenu de ce site est disponible sous
                    <a href="https://github.com/etalab/licence-ouverte/blob/master/LO.md" target="_blank">
                        Licence Ouverte 2.0 <i class="fa fa-external-link" aria-hidden="true"></i>
                    </a>
                </p>
            </div>
        </div>
    </div>
</footer>
```

### CSS back-office (Feuille de Style)
```css
/* FOOTER */
.ods-front-footer__back {
    position: relative;
    width: 100%;
    height: auto;
    padding-bottom: 1%;
    padding-top: 1em;
    border-top: solid 2px var(--foot-border);
    background-color: white;
}
.ods-front-footer__container {
    max-width: var(--general-width);
    margin-top: 20px;
    margin-left: auto;
    margin-right: auto;
}
.ods-front-footer__body {
    display: flex;
    flex-direction: row;
    align-items: center;
    flex-wrap: wrap;
}
.ods-front-footer__logo {
    height: 80px;
    width: auto;
}
.ods-front-footer__external-links {
    margin-left: auto;
}
.ods-front-footer__external-links a {
    color: var(--grey-50);
}
.ods-front-footer__bottom-list {
    border-top: solid 2px var(--foot-border);
    display: flex;
    flex-flow: wrap;
    align-items: center;
    padding: 0;
    margin-bottom: 0;
}
.ods-front-footer__bottom-item {
    color: grey;
    display: inline;
}
.ods-front-footer__link {
    line-height: 40px;
    padding-left: 1em;
    padding-right: 1em;
}
.ods-front-footer__licence {
    text-align: left;
    padding-left: 1em;
}

@media (max-width: 991px) {
    .ods-front-footer__body {
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
    }
    .ods-front-footer__external-links {
        margin-left: 0;
    }
}
```

---

## 6. Pattern B — Footer DSFR complet (ex: data.education.gouv.fr)

Pattern pour les portails gouvernementaux avec conformité DSFR.

### HTML back-office
```html
<footer class="fr-footer" role="contentinfo" id="footer-portail">
    <div class="fr-container">
        <!-- Zone haute : logo RF + liens institutionnels -->
        <div class="fr-footer__body">
            <div class="fr-footer__brand fr-enlarge-link">
                <a href="/" title="Retour à l'accueil - Nom du ministère">
                    <p class="fr-logo">
                        Gouvernement
                    </p>
                </a>
            </div>
            <div class="fr-footer__content">
                <ul class="fr-footer__content-list">
                    <li class="fr-footer__content-item">
                        <a class="fr-footer__content-link" target="_blank" href="https://www.info.gouv.fr">info.gouv.fr</a>
                    </li>
                    <li class="fr-footer__content-item">
                        <a class="fr-footer__content-link" target="_blank" href="https://service-public.fr">service-public.fr</a>
                    </li>
                    <li class="fr-footer__content-item">
                        <a class="fr-footer__content-link" target="_blank" href="https://legifrance.gouv.fr">legifrance.gouv.fr</a>
                    </li>
                    <li class="fr-footer__content-item">
                        <a class="fr-footer__content-link" target="_blank" href="https://data.gouv.fr">data.gouv.fr</a>
                    </li>
                </ul>
            </div>
        </div>
        <!-- Zone basse : liens légaux + gestion cookies + copyright -->
        <div class="fr-footer__bottom">
            <ul class="fr-footer__bottom-list">
                <li class="fr-footer__bottom-item">
                    <a class="fr-footer__bottom-link" href="/pages/plan-site">Plan du site</a>
                </li>
                <li class="fr-footer__bottom-item">
                    <a class="fr-footer__bottom-link" href="/pages/accessibilite">Accessibilité</a>
                </li>
                <li class="fr-footer__bottom-item">
                    <a class="fr-footer__bottom-link" target="_blank" href="/terms/terms-and-conditions">Mentions légales</a>
                </li>
                <li class="fr-footer__bottom-item">
                    <a class="fr-footer__bottom-link" href="/terms/cookies-policy/">Données personnelles</a>
                </li>
                <li class="fr-footer__bottom-item">##manage-cookies##</li>
            </ul>
            <div class="fr-footer__bottom-copy">
                <p>Sauf mention contraire, tous les contenus de ce site sont sous
                    <a href="https://github.com/etalab/licence-ouverte/blob/master/LO.md" target="_blank">licence etalab-2.0</a>
                </p>
            </div>
        </div>
    </div>
</footer>
```

### Points clés DSFR
- Le logo RF **en texte** (`<p class="fr-logo">Gouvernement</p>`) est rendu visuellement par le CSS DSFR — ne pas remplacer par une image
- `fr-enlarge-link` sur `.fr-footer__brand` étend la zone cliquable au bloc entier
- Les séparateurs verticaux entre items `.fr-footer__bottom-item` sont générés automatiquement via `::before` en CSS DSFR
- `##manage-cookies##` s'insère dans un `<li class="fr-footer__bottom-item">` — le style `.ods-manage-cookies-preferences__show-button` est prévu pour cohabiter avec `.fr-footer__bottom-link`

### CSS complémentaire back-office (si nécessaire)
```css
/* Ajustement manage-cookies dans contexte DSFR */
.ods-manage-cookies-preferences__show-button {
    font-size: 0.75rem;
    line-height: 1.25rem;
    color: var(--text-mention-grey);
}

/* Wrap umami ou autre tracking wrapper */
.umami-footer {
    display: flex;
    width: 100%;
    flex-wrap: wrap;
}
```

---

## 7. Pattern C — Footer full custom avec logos partenaires (ex: CNAF)

Pattern quand on veut **un design entièrement sur-mesure** : logos multiples, réseaux sociaux, liens directs (sans placeholders `##legal##` / `##manage-cookies##`).

### HTML back-office
```html
<div class="ods-front-footer">
    <div class="container footer-container">
        <!-- Logos partenaires -->
        <div class="footer-logo-container">
            <div class="logo-container">
                <a target="_blank" href="https://www.partenaire1.fr/" class="logo-text">
                    <img ng-non-bindable class="logo-footer" src="/assets/theme_image/logo-partenaire1.png"/>
                </a>
            </div>
            <div class="logo-container">
                <a target="_blank" href="https://www.partenaire2.fr/" class="logo-text">
                    <img ng-non-bindable class="logo-footer" src="/assets/theme_image/logo-partenaire2.png"/>
                </a>
            </div>
            <!-- Réseau social -->
            <div class="logo-container">
                <a target="_blank" href="https://www.linkedin.com/..." class="rs-group">
                    <img ng-non-bindable class="rs-picto" src="/assets/theme_image/logo-linkedin.png"/>
                </a>
            </div>
        </div>
        <!-- Liens légaux directs (sans placeholders) -->
        <div class="liens-utile">
            <a target="_blank" href="/terms/terms-and-conditions/">Conditions d'utilisation</a>
            <a target="_blank" href="/terms/privacy-policy/">Politiques de confidentialité</a>
            <a target="_blank" href="/terms/cookies-policy/">Gestion des cookies</a>
            <a target="_blank" href="/pages/contact/">Contact</a>
        </div>
    </div>
</div>
```

### CSS back-office
```css
/* FOOTER */
.ods-front-footer {
    height: 100%;
    font-size: 15px;
    padding: 0.5rem;
    background-color: var(--primary); /* couleur principale du portail */
}
.footer-container {
    display: flex;
    flex-direction: row;
    gap: 80px;
}
.footer-logo-container {
    display: flex;
    flex-direction: row;
    gap: 20px;
    justify-content: space-around;
    align-items: center;
}
.logo-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
}
.logo-footer {
    height: 50px;
    width: auto;
}
.rs-picto {
    height: 30px;
}
.liens-utile {
    display: flex;
    gap: 20px;
    padding-top: 15px;
    flex-wrap: wrap;
    margin-bottom: 20px;
}
.liens-utile a {
    color: white; /* sur fond coloré */
}

@media (max-width: 991px) {
    .footer-container {
        flex-direction: column;
        gap: 10px;
    }
}
```

### ⚠️ Attention : liens directs vs placeholders
Dans ce pattern, les liens légaux sont **codés en dur** dans le HTML (URLs absolues). Cela signifie :
- Les URLs doivent être mises à jour manuellement si elles changent
- `##manage-cookies##` n'est **pas utilisé** — le lien cookies est un `<a>` statique vers `/terms/cookies-policy/`
- Ce pattern peut poser des problèmes si la bannière cookies doit être pilotée dynamiquement

> Recommandation : préférer `##manage-cookies##` dans un `<li>` pour que la bannière soit correctement gérée par la plateforme.

---

## 8. Comparatif des patterns

| | Pattern A — ODS natif | Pattern B — DSFR | Pattern C — Full custom |
|---|---|---|---|
| **Exemple** | equipements.sports.gouv.fr | data.education.gouv.fr | cnaf.huwise.com |
| **Classes** | `.ods-front-footer__*` | `.fr-footer__*` | Classes custom |
| **Logo** | Image custom | Logo RF texte (`fr-logo`) | Images custom + réseaux sociaux |
| **Liens légaux** | `##legal##` + `##manage-cookies##` dans `<li>` | Liens directs + `##manage-cookies##` dans `<li>` | Liens directs codés en dur |
| **Responsive** | CSS media queries | CSS DSFR natif | CSS media queries custom |
| **Font Awesome** | Oui (icônes external-link) | Non | Optionnel |
| **Recommandé pour** | Portails ODS standard | Portails gouvernementaux | Portails très brandés |

---

## 9. Checklist footer

- [ ] Le footer est configuré dans **Portail > Style > Pied de page** (pas dans le code éditeur des pages)
- [ ] Le CSS du footer est dans **Portail > Style > Feuille de Style**
- [ ] `##manage-cookies##` est présent si une bannière cookies est configurée
- [ ] `ng-non-bindable` est ajouté sur les `<img>` dont le `src` contient des `{}`
- [ ] Pas de JavaScript dans le footer (interdit par la plateforme)
- [ ] Le thème a été prévisualisé avant mise en ligne
- [ ] Les contrastes de couleurs respectent WCAG AA
- [ ] Les liens externes ont `target="_blank"`
- [ ] Les logos sont uploadés dans **Portail > Images et polices** et référencés via `/assets/theme_image/`
- [ ] Le responsive est géré en CSS (pas de `ods-responsive-menu` dans le footer)
- [ ] Si liens légaux codés en dur : les URLs sont correctes et maintenues à jour
- [ ] Si DSFR : `.ods-manage-cookies-preferences__show-button` est stylistiquement cohérent avec `.fr-footer__bottom-link`

---
