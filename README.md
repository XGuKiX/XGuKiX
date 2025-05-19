## Hi there 👋
// Timeswap === // Frontend : React Native + Expo // Backend : Node.js (Express) + Supabase Edge Functions + Stripe // Données : Import des coiffeurs & esthéticiennes via API + géocodage OpenStreetMap // Authentification sécurisée avec validation stricte du mot de passe

// === 1. BACKEND : Node.js Express + Stripe Webhooks === // install: express, stripe, supabase-js, dotenv, body-parser, zxcvbn

/* server.js */ require('dotenv').config(); const express = require('express'); const app = express(); const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY); const bodyParser = require('body-parser'); const zxcvbn = require('zxcvbn'); const { createClient } = require('@supabase/supabase-js');

const supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_SERVICE_ROLE_KEY); app.use(bodyParser.json());

// Authentification personnalisée avec validation mot de passe app.post('/signup', async (req, res) => { const { email, password, username } = req.body; const strong = /^(?=.[a-z])(?=.[A-Z])(?=.\d)(?=.[\W_]).{12,}$/; if (!strong.test(password)) { return res.status(400).json({ error: 'Mot de passe trop faible (12 caractères, 1 maj, 1 chiffre, 1 caractère spécial)' }); }

const { data, error } = await supabase.auth.signUp({ email, password, options: { data: { username } }, });

if (error) return res.status(400).json({ error: error.message }); res.status(200).json({ user: data.user }); });

// Stripe Webhook pour l’abonnement app.post('/webhook', bodyParser.raw({ type: 'application/json' }), async (req, res) => { const sig = req.headers['stripe-signature']; let event; try { event = stripe.webhooks.constructEvent(req.body, sig, process.env.STRIPE_WEBHOOK_SECRET); } catch (err) { return res.status(400).send(Webhook Error: ${err.message}); }

if (['customer.subscription.updated', 'customer.subscription.created'].includes(event.type)) { const sub = event.data.object; const email = sub.metadata.email; await supabase.from('users').update({ abonnement: 'premium' }).eq('email', email); }

if (event.type === 'customer.subscription.deleted') { const sub = event.data.object; const email = sub.metadata.email; await supabase.from('users').update({ abonnement: 'gratuit' }).eq('email', email); }

res.json({ received: true }); });

app.listen(4242, () => console.log('Serveur backend en écoute sur port 4242'));

// === 2. Edge Function Supabase : import des professionnels === /* import { serve } from 'https://deno.land/std/http/server.ts'; import { createClient } from 'https://esm.sh/@supabase/supabase-js';

serve(async (req) => { const supabase = createClient(Deno.env.get('SUPABASE_URL')!, Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!); const response = await fetch('https://api.annuaire-sante.fr/api/professionnels?specialite=coiffure,esthetique'); const liste = await response.json();

for (const item of liste) { const geo = await fetch(https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(item.adresse)}); const coord = await geo.json(); if (!coord.length) continue; await supabase.from('professionnels').insert({ type: item.specialite, nom: item.nom, adresse: item.adresse, ville: item.ville, departement: item.departement, telephone: item.telephone, latitude: coord[0].lat, longitude: coord[0].lon, }); }

return new Response(JSON.stringify({ status: 'ok' }), { status: 200 }); }); */

// === 3. FRONTEND React Native (inchangé, voir message précédent) === // Ajout sur écran d'inscription d’un champ username et de la validation regex sur mot de passe

// === 4. Fichier .env Exemple === /* SUPABASE_URL=https://xxxxx.supabase.co SUPABASE_SERVICE_ROLE_KEY=xxxxx STRIPE_SECRET_KEY=sk_test_xxx STRIPE_WEBHOOK_SECRET=whsec_xxx */

<!--
**XGuKiX/XGuKiX** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
