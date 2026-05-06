# ContentCraft — Guide de déploiement complet

## Ce que tu as
Une application web SaaS complète de génération de contenu IA :
- Interface professionnelle (dark mode, animations, responsive)
- Générateur fonctionnel (LinkedIn, produit, email, pub, Twitter)
- Système de limitation (3 générations gratuites puis paywall)
- Modal de paiement prêt à connecter à Stripe
- 3 plans tarifaires (Gratuit / Pro 19€ / Business 49€)

---

## ÉTAPE 1 — Configurer l'API Claude

Dans `index.html`, la ligne suivante appelle l'API Anthropic :
```javascript
const response = await fetch('https://api.anthropic.com/v1/messages', {
```

**PROBLÈME IMPORTANT** : En l'état, la clé API est exposée dans le code frontend.
Pour un vrai produit, tu dois créer un backend (voir Étape 3).

Pour tester rapidement : va sur https://console.anthropic.com → API Keys → créer une clé.

---

## ÉTAPE 2 — Déployer sur Vercel

1. Crée un compte sur https://vercel.com (gratuit)
2. Installe Vercel CLI : `npm install -g vercel`
3. Dans le dossier `contentcraft/` : `vercel deploy`
4. Ou importe directement depuis GitHub : vercel.com/new

Ton site sera en ligne en 2 minutes sur une URL type `contentcraft.vercel.app`.

---

## ÉTAPE 3 — Backend sécurisé (OBLIGATOIRE pour la production)

Pour ne pas exposer ta clé API, crée une API route Vercel :

Crée `/api/generate.js` :
```javascript
export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).end();

  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify(req.body)
  });

  const data = await response.json();
  res.json(data);
}
```

Dans Vercel Dashboard → Settings → Environment Variables :
- `ANTHROPIC_API_KEY` = ta clé Anthropic

Dans `index.html`, remplace l'URL :
```javascript
// Avant (dangereux)
fetch('https://api.anthropic.com/v1/messages', ...)

// Après (sécurisé)
fetch('/api/generate', ...)
// Et retire le header x-api-key du frontend
```

---

## ÉTAPE 4 — Configurer Stripe

1. Crée un compte sur https://stripe.com
2. Dashboard → Produits → Créer deux produits :
   - **ContentCraft Pro** : abonnement 19€/mois
   - **ContentCraft Business** : abonnement 49€/mois
3. Pour chaque produit, copie le lien de paiement (Payment Links)
4. Dans `index.html`, remplace :
```javascript
const stripeLinks = {
  pro: 'https://buy.stripe.com/VOTRE_LIEN_PRO',       // ← colle ici
  business: 'https://buy.stripe.com/VOTRE_LIEN_BUSINESS' // ← colle ici
};
```

Stripe gère tout : paiement, abonnements, emails de confirmation, facturation.

---

## ÉTAPE 5 — Domaine personnalisé

Sur Vercel → Settings → Domains → ajouter ton domaine.
Domaine pas cher : https://namecheap.com (~10€/an)

---

## Coûts à prévoir

| Service | Coût |
|---------|------|
| Vercel (hébergement) | Gratuit (plan hobby) |
| Domaine | ~10€/an |
| Anthropic API | ~0.003€ / génération |
| Stripe | 1.4% + 0.25€ / transaction |

→ Si un utilisateur Pro (19€/mois) fait 200 générations, ça te coûte ~0.60€ en API.
→ Marge brute : ~18€ par utilisateur Pro par mois (hors Stripe fees).

---

## Pour aller plus loin

- **Authentification** : Supabase (gratuit jusqu'à 50k utilisateurs) — https://supabase.com
- **Base de données** (historique des générations) : Supabase PostgreSQL
- **Analytics** : Plausible ou Umami (privacy-friendly)
- **Email transactionnel** : Resend.com (gratuit jusqu'à 3k emails/mois)

---

## Réalité du business

Ce site est une base solide. Ce qu'il ne fait pas encore :
- Authentification réelle (comptes utilisateurs)
- Vérification que l'utilisateur a bien payé avant de débloquer l'accès
- Stockage de l'historique des générations

Ces fonctionnalités nécessitent un backend complet (Supabase + Next.js recommandés).
Le plus dur reste d'acquérir des clients — le code est le moindre de tes problèmes à terme.
