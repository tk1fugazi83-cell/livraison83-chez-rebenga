# LIVRAISON83 CHEZ REBENGA — intégration Supabase

Cette version relie la Mini App Telegram à la fonction Edge `order-api` du projet Supabase `lgmiggbvzcuynmhpsapo`. Les commandes ne sont plus des exemples : elles sont envoyées au serveur et le client ne peut lire que ses commandes Telegram après validation serveur de `Telegram.WebApp.initData`.

## Fonctions incluses
- Création d'une demande de livraison réelle depuis `index.html`.
- Historique et suivi du statut depuis la base Supabase.
- Tableau d'administration dans `admin.html` (pas d'interface livreur).
- Connexion admin par e-mail/mot de passe Supabase Auth et contrôle serveur via la table `admin_users`.
- Prix laissé « À confirmer » jusqu'à ce qu'un administrateur définisse le montant.

## 1. Lancer et compiler

```bash
npm install
npm run dev
npm run build
```

La sortie de production est `dist/`. Pour Telegram, l'application doit être publiée sur une URL HTTPS et configurée dans BotFather.

## 2. Autoriser un compte administrateur

1. Dans Supabase Dashboard → Authentication → Users, créez/invitez votre compte administrateur et définissez son mot de passe.
2. Copiez l'UUID de cet utilisateur.
3. Dans SQL Editor, exécutez la requête ci-dessous en remplaçant l'UUID par celui de votre compte :

```sql
insert into public.admin_users (user_id)
values ('UUID-DE-VOTRE-COMPTE-AUTH')
on conflict (user_id) do nothing;
```

4. Publiez le dossier `dist/` sur votre hébergeur HTTPS. Ouvrez `https://VOTRE-DOMAINE/admin.html` pour administrer les commandes.

La page d'administration n'offre pas d'inscription publique. Un compte Auth sans entrée dans `admin_users` est refusé par l'API.

## 3. Configuration API

Les paramètres publics de l'application se trouvent dans `src/config.js`. La clé `sb_publishable_...` est une clé publique prévue pour le navigateur. **N'y placez jamais la clé `service_role` ou un secret.** Le token du bot Telegram doit rester uniquement dans les secrets Supabase Edge Functions sous le nom `TELEGRAM_BOT_TOKEN`.

La fonction Edge `order-api` doit être active et conserver `verify_jwt=false`, car les clients Telegram s'authentifient avec la signature de `initData`; les actions d'administration valident séparément le JWT Supabase et l'autorisation `admin_users`.

## 4. Limites à régler avant ouverture publique

- Remplacer le lien `https://t.me/` du bouton de contact dans `src/main.js` par le compte officiel du service client.
- Restreindre `Access-Control-Allow-Origin` dans la fonction Edge à votre domaine HTTPS une fois l'URL d'hébergement connue.
- Effectuer un test complet dans Telegram avec une vraie session utilisateur et une commande test.
- Définir la politique de confidentialité, la conservation des données et les modalités de confirmation du prix.
- Cette intégration gère les commandes et leurs statuts, pas le paiement en ligne ni le GPS temps réel.
