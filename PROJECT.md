# SénégalRek — état du projet

Ce document décrit le projet, les décisions prises et l'état actuel du thème. Il complète `CLAUDE.md` (qui contient les instructions techniques pour travailler sur ce thème Liquid) sans le remplacer.

## Le projet

SénégalRek est une boutique Shopify (thème basé sur Dawn) qui regroupe des produits de **plusieurs marques européennes** et les livre au Sénégal, avec une vraie expérience e-commerce : prix fixes, paiement en ligne, livraison suivie — explicitement **pas** un service de "devis + achat pour compte" informel.

**Cible** : la diaspora sénégalaise en Europe (achats pour la famille au pays) et les clients au Sénégal.

**Positionnement** : doit paraître sérieux et professionnel (prix clairs, délais de livraison affichés, paiement en ligne) plutôt qu'improvisé. Les signaux de confiance comptent plus que l'étendue du catalogue à ce stade.

**Vision produit actuelle** : une page par marque, navigable via des onglets — rail latéral sticky sur desktop, bandeau à défilement horizontal sur mobile — chaque page marque reprenant les couleurs de cette marque. Le reste (fiche produit, panier, checkout) garde la forme d'une boutique en ligne classique.

## Architecture retenue

Deux principes structurent toutes les décisions techniques prises jusqu'ici :

1. **S'appuyer sur les mécanismes natifs de Shopify** plutôt que d'inventer de la logique Liquid custom pour résoudre "quelle marque pour ce produit/cette page ?". Concrètement : chaque marque = une **Collection**, et la couleur de marque passe par les **color schemes** natifs + des **templates alternatifs** (`collection.brand-a.json`, `product.brand-a.json`, etc.), assignés manuellement dans l'Admin. Pas de metafields, pas de matching par `vendor`.
2. **Le thème fournit des sections/blocks ; le marchand assemble** via l'éditeur de thème et l'Admin (créer les collections, assigner les templates, lier les blocs). C'est un choix délibéré pour rester simple et robuste sans connexion à une boutique live pendant le développement.

## Ce qui est construit (code, thème)

### Navigation multi-marques
- `sections/brand-nav-group.json` + `sections/brand-nav.liquid` — rail de navigation entre marques, affiché **sur tout le site** (pas seulement les pages boutique), via un groupe de sections custom rendu dans `layout/theme.liquid`. Sticky à gauche sur desktop (≥990px), bandeau scrollable en haut sur mobile. 3 emplacements pré-configurés ("Marque A/B/C", color schemes 6/7/8), à lier à de vraies collections dans l'éditeur de thème.
- `layout/theme.liquid` — restructuré pour placer le rail en colonne à côté de `<main>` (grille CSS `.brand-shell`). Le contenu principal est épinglé explicitement à la 2ᵉ colonne pour ne pas se retrouver écrasé dans la colonne étroite si le rail est vide.

### Page d'accueil
- `templates/index.json` reconstruit avec du vrai contenu français : hero → vitrine des 3 marques → "Comment ça marche" (3 étapes) → sélection produits → bandeau d'appel.
- Nouvelles sections : `sections/hero-route.liquid` (titre + texte + 2 boutons + repère graphique Europe → Sénégal), `sections/brand-showcase.liquid` (cartes marques avec swatch de couleur réel par marque), `sections/how-it-works.liquid` (étapes numérotées automatiquement), `sections/cta-band.liquid` (bandeau de clôture).
- Conçue pour fonctionner **sans photographie produit** (aucune vraie photo dans le thème actuellement) : typographie et couleur portent le design, pas d'image.

### Identité visuelle
- Police de titres : **Montagu Slab** (`montagu_slab_n5`). Police de texte/UI : **Archivo** (`archivo_n4`). Remplace Assistant (police générique utilisée par défaut sur Dawn).
- Palette : inchangée par rapport au premier rendu (ivoire `#FBF7F1`, vert profond `#1B4332`, terracotta `#C1652F`, brun foncé `#241C15`, beige `#F2E9DC`), plus 3 nouveaux color schemes placeholder pour les marques démo :
  - `scheme-6` "Atelier Bleu" — `#EDF1F5` / `#2C4A6E`
  - `scheme-7` "Rose Poudré" — `#F3E8EE` / `#7A3B57`
  - `scheme-8` "Vert Sauge" — `#EEF0E4` / `#5B6B4D`

### Templates alternatifs prêts à assigner
- `templates/collection.brand-a.json`, `.brand-b.json`, `.brand-c.json`
- `templates/product.brand-a.json`, `.brand-b.json`, `.brand-c.json`

Chacun reprend le template par défaut avec le `color_scheme` basculé sur 6/7/8.

### Historique des commits (session actuelle)
```
64a59e9 Remove the Paiement/Livraison/Suivi info card from the hero
f9ac78b Fix homepage layout collapsing into the narrow rail column
c2d6d11 Redesign homepage with real content and a distinctive type pairing
aea7381 Add multi-brand navigation rail with per-brand color schemes
2fa18df First render: SénégalRek brand pass on Dawn (session précédente)
```

## État actuel — ce qui manque encore

Le thème est **à jour côté code et validé** (`shopify theme check` : 0 erreur), mais rien n'est fonctionnel côté catalogue tant que ces étapes manuelles n'ont pas été faites dans l'Admin :

- [ ] Créer 3 collections démo (une par marque placeholder), y ajouter 1-2 produits chacune
- [ ] Assigner à chaque collection son template alternatif (`brand-a`/`brand-b`/`brand-c`)
- [ ] Assigner aux produits démo leur template alternatif correspondant
- [ ] Dans l'éditeur de thème, lier les 3 blocs de la section "Marques" (le rail) aux 3 collections créées

**Point d'attention** : le thème actuel est un **thème brouillon, pas encore publié**. Le sélecteur de template alternatif sur la fiche Collection/Produit dans l'Admin ne liste que les templates du thème **publié** — pour un thème brouillon, il faut choisir le template alternatif depuis l'éditeur de thème (Personnaliser) de ce thème précis, pas depuis la fiche Collection/Produit.

Une fois ces étapes faites, le rail change bien de couleur en cliquant entre marques, et la fiche produit garde la couleur de la marque (grâce à son template alternatif) — mais ceci reste un travail manuel par produit, il n'y a pas de résolution automatique "ce produit appartient à quelle marque" dans le code.

## Problèmes connus — à traiter en prochaine session

- **Le bug "page écrasée à gauche" n'est pas résolu sur PC.** Le fix du commit `f9ac78b` (épingler `<main>` en colonne 2 de la grille `.brand-shell`) corrige l'affichage mobile, mais **sur desktop le problème persiste**. À reprendre : vérifier pourquoi la grille à 2 colonnes ne se comporte pas comme prévu sur les grands écrans — probablement lié au point suivant (le rail ne se rend peut-être pas du tout, ce qui fausse la grille même avec le fix appliqué).
- **Le rail de navigation entre marques ne s'affiche pas sur les pages marque.** En l'état, la seule façon d'accéder aux marques est la vitrine de cartes (`brand-showcase`) sur la page d'accueil — ce qui fonctionne bien *pour la page d'accueil*, mais une fois sur la page dédiée à une marque (collection), il n'y a **pas d'onglets de navigation scrollables** en haut/à côté comme demandé initialement (rail sticky sur desktop, bandeau scrollable sur mobile, visible sur tout le site). Le rail (`sections/brand-nav.liquid` + `brand-nav-group.json`) est codé mais ne se manifeste visiblement nulle part en dehors du code — à déboguer : voir si le groupe de sections custom se rend réellement, ou s'il faut revoir l'approche (ex. type de groupe, placement dans `layout/theme.liquid`).

## Décisions ouvertes / à trancher plus tard

- Les 3 marques actuelles (A/B/C, schemes 6/7/8) sont des **placeholders**. À remplacer par de vraies marques (nom, univers, couleurs) dès qu'elles sont connues — ça implique de renommer les color schemes dans l'éditeur et de remplacer les blocs du rail + de la vitrine.
- Aucune vraie photo produit/lifestyle dans le thème — la page d'accueil a été conçue pour fonctionner sans, mais elle a des emplacements prévus pour en recevoir plus tard sans tout refaire.
- Délai de livraison affiché ("10 à 15 jours ouvrés") est un placeholder en attente de vrais chiffres.
- Le thème n'est pas publié — à publier quand le catalogue démo (ou réel) sera en place et validé visuellement.
