# Hoto

**Hoto** est une boîte Gmail qui permet de recevoir de l'argent partout dans le monde et de retirer par Mobile Money.
Couleurs : rouge `#B3261E`, blanc, kaki `#C3B091` / `#6B5F3A`.

- Compte en 1 clic avec votre compte Google (Gmail), photo de profil reprise automatiquement.
- Un lien de paiement personnel : `https://votre-site/pay/votre-nom`.
- Chaque paiement confirmé arrive aussi par e-mail dans votre boîte Gmail.
- Paiements acceptés depuis l'Afrique, les Amériques, l'Europe, l'Asie et l'Océanie.
- Retrait vers Mobile Money protégé par un code PIN.

> Gmail ne déplace pas l'argent. L'argent passe par **Stripe** et **Flutterwave** ; Gmail sert à la connexion et aux notifications.

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `hoto-complet.html` | Frontend complet en un fichier (HTML + CSS + JavaScript), à placer dans `public/index.html` |
| `server.js` | Backend Node.js (Google OAuth, Gmail, Stripe, Flutterwave, base SQLite) |
| `public/index.html` | Emplacement du frontend servi par le backend (copie de `hoto-complet.html`) |
| `test.js` | Tests locaux (signature de webhook, crédit unique, solde) |
| `.env.example` | Modèle des variables d'environnement |
| `package.json` | Dépendances (Express) |

## Installation

### 1. Installer
```
npm install
cp .env.example .env
```
Node.js 22.13 ou plus récent. Le fichier `public/index.html` est déjà fourni ; si vous modifiez `hoto-complet.html`, copiez-le dans `public/index.html`.

### 2. Google (connexion et Gmail)
1. Sur console.cloud.google.com, créez un projet et activez **Gmail API**.
2. Configurez l'écran de consentement OAuth, puis créez un ID client **Application Web**.
3. URI de redirection autorisée : `BASE_URL/auth/google/callback`.
4. Renseignez `GOOGLE_CLIENT_ID` et `GOOGLE_CLIENT_SECRET`.

Le scope `gmail.send` est sensible : en production, Google demande une vérification de l'application.

### 3. Stripe (Amériques, Europe, Asie, Océanie)
1. Renseignez `STRIPE_SECRET_KEY`.
2. Créez un webhook vers `BASE_URL/webhooks/stripe` pour `checkout.session.completed` et `checkout.session.async_payment_succeeded`.
3. Renseignez `STRIPE_WEBHOOK_SECRET`.

### 4. Flutterwave (Afrique : Mobile Money et carte)
1. Renseignez `FLW_SECRET_KEY`.
2. Créez un webhook vers `BASE_URL/webhooks/flutterwave` avec un « secret hash » de votre choix, à mettre dans `FLW_SECRET_HASH`.
3. Activez les moyens de paiement et les transferts dans votre compte.

### 5. Lancer
```
node --env-file=.env server.js
```
Ouvrez `BASE_URL` (par défaut `http://localhost:3000`).
En production : déployez derrière **HTTPS** (Render, Railway, VPS), définissez `NODE_ENV=production` et un `SESSION_SECRET` long, et sauvegardez `hoto.db`.

### Variables d'environnement

| Variable | Description |
|---|---|
| `BASE_URL` | Adresse publique du site |
| `SESSION_SECRET` | Chaîne aléatoire longue (obligatoire en production) |
| `HOTO_FEE_PCT` | Frais Hoto en % (défaut 1) |
| `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | Connexion Google et Gmail |
| `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` | Paiements par carte dans le monde |
| `FLW_SECRET_KEY`, `FLW_SECRET_HASH` | Paiements et retraits en Afrique |

## Devises
- **Flutterwave** : XOF, XAF, GHS, KES, NGN, UGX, TZS, RWF, ZAR, ZMW.
- **Stripe** : USD, EUR, GBP, CAD, BRL, MXN, INR, JPY, AUD, NZD, SGD, HKD, CHF, SEK, NOK, DKK, AED.

Les devises et moyens de paiement réellement disponibles dépendent du pays de vos comptes Stripe et Flutterwave.

## Sécurité
- Signatures des webhooks vérifiées, paiements revérifiés auprès du fournisseur.
- Un paiement n'est jamais crédité deux fois.
- Montants stockés en entiers (unités mineures).
- PIN haché (scrypt), blocage 15 minutes après 3 erreurs.
- Cookie de session signé, protection CSRF, vérification du solde avant retrait.

## Tests
```
npm test
```

## Avant d'ouvrir au public
- Recevoir et transférer de l'argent pour des tiers est **réglementé** (licence ou partenariat de paiement, vérification d'identité, lutte contre le blanchiment, protection des données). Consultez un juriste de votre pays.
- Pour les retraits, il faut le code opérateur Flutterwave de chaque pays (voir leur documentation).
- Chiffrez les jetons Gmail stockés et passez à PostgreSQL pour la production.
- Ajoutez conditions d'utilisation et politique de confidentialité (exigées par Google et les fournisseurs de paiement).

## Licence
À définir par le propriétaire du projet.
