# Instructions pour le développement du site Laure Ferrando

## Architecture du projet

**Stack technique**: Astro SSR (mode standalone) + Vue 3 pour l'interactivité + WooCommerce REST API

### Structure hybride Astro + Vue
- **Composants Astro** (`src/components/*.astro`): Pages statiques, Header, Footer, BurgerButton
- **Composants Vue** (`src/components/reactive/*.vue`): Toutes les fonctionnalités interactives (panier, formulaires, carrousels)
- **Pattern d'intégration**: Les pages Astro importent les composants Vue avec `client:load` pour l'hydratation côté client

### État global avec Nanostores
**Fichier central**: `src/utils/store.js`
- Utilise `nanostores` pour la gestion d'état réactive partagée entre composants Vue
- Atoms principaux: `cart`, `weight`, `total`, `subTotal`, `shipping`, `totalItems`
- **Persistance**: Le panier est automatiquement sauvegardé dans `localStorage` via `storeLocalSession()`
- **Pattern de mise à jour**: Toujours appeler `updateTotals()` après modification du cart, puis `storeLocalSession()`
- **Initialisation**: `initializeStore()` récupère le panier depuis localStorage au chargement

### Intégration WooCommerce
**API client**: `src/utils/fetchUtils.js` - classe `Fetcher`
- **Authentification**: Basic Auth via variables d'environnement `PUBLIC_CLIENT_KEY` et `PUBLIC_CLIENT_SECRET`
- **Endpoint API**: `PUBLIC_API_URL` (ex: https://admin.laure-ferrando.com)
- Les produits sont récupérés via `wp-json/wc/v3/products`
- **Pattern**: Toujours instancier `new Fetcher()` puis appeler `fetchData(endpoint)`
- **Gestion d'erreur**: Vérifier `!Array.isArray(data) || data.length === 0` pour détecter les produits inexistants

### Routes et pages dynamiques
- **Boutique**: `/boutique/[product].astro` utilise le slug produit pour fetch depuis WooCommerce
- **Pages légales**: `/[legales]/index.astro` avec `export const prerender = true` pour pré-rendu statique
- **Commandes**: `/order/[order].astro` pour la confirmation de commande

## Conventions de code

### Gestion du panier (Cart)
**Ajouter un article**:
```javascript
addCartItem(id, name, image, price, weight)
```
- `weight` doit être un Number (converti automatiquement si String)
- Le poids par défaut est 0 si non fourni
- La quantité s'incrémente si l'article existe déjà

**Calcul des frais de port** (logique dupliquée dans `store.js` et `Cart.vue`):
- < 1kg: 5€
- 1-5kg: 10€
- > 5kg: 15€

**Important**: Le poids des produits WooCommerce peut être `null` - toujours vérifier et définir à `0` par défaut

### Composants Vue interactifs
Pattern standard d'utilisation dans Astro:
```astro
<Cart client:load />
<AddToCartButton client:load id={item.id} name={item.name} ... />
```
- `client:load`: Hydratation immédiate (utilisé pour panier, formulaires)
- Les props sont passées en HTML attributes (automatiquement typés)

### Variables CSS globales
Définies dans `src/layouts/Layout.astro`:
```css
--font-family-titles: "Libre Baskerville", serif;
--font-family-text: "Lato", sans-serif;
--background-color: #f3f1ed;
--text-color: #342d1f;
--accent-color: #b39966;
--large-screen: 992px;
```

## Commandes de développement

```bash
bun run dev          # Dev server sur localhost:4321
bun run host         # Dev server accessible sur le réseau local
bun run build        # Build production
bun run preview      # Preview du build
```

**Déploiement Docker**:
```bash
docker-compose up -d  # Lance le conteneur sur port 4321
```

## Variables d'environnement

Créer un fichier `.env` à la racine (voir `.env.example`):
```
PUBLIC_CLIENT_KEY=ck_...
PUBLIC_CLIENT_SECRET=cs_...
PUBLIC_API_URL=https://admin.laure-ferrando.com
```

## Détails techniques importants

- **Mode SSR**: Le projet utilise `output: "server"` dans astro.config.mjs - nécessaire pour les routes dynamiques
- **Adapter Node**: Mode `standalone` pour déploiement autonome
- **Fontes**: Import via `@fontsource` dans Layout.astro
- **Script SMTP**: Chargé depuis `smtp.js` CDN (header du Layout) pour le formulaire de contact
- **Langue**: Attribut `lang="fr"` dans Layout.astro
