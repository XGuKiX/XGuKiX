## Hi there 👋
git clone https://github.com/XGuKiX/NOM_DU_DEPOT.git
cd TIMESWAP 
// === TIMESWAP — Application complète === // Frontend : React Native (Expo) // Backend : Node.js + Supabase Edge Function + Stripe // Fonctionnalités : Prise de rendez-vous, échange, chat, géolocalisation, paiement, abonnement

// === 1. BACKEND Node.js (server.js) === require('dotenv').config(); const express = require('express'); const bodyParser = require('body-parser'); const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY); const { createClient } = require('@supabase/supabase-js');

const app = express(); const supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_SERVICE_ROLE_KEY);

app.use(bodyParser.json());

// Authentification sécurisée app.post('/signup', async (req, res) => { const { email, password, username } = req.body; const pattern = /^(?=.[A-Z])(?=.[0-9])(?=.[!@#$%^&()_+-={}:";'<>?,./]).{12,}$/; if (!pattern.test(password)) { return res.status(400).json({ error: 'Mot de passe invalide (12+ caractères, 1 majuscule, 1 chiffre, 1 spécial)' }); }

const { data, error } = await supabase.auth.signUp({ email, password, options: { data: { username } }, });

if (error) return res.status(400).json({ error: error.message }); res.status(200).json({ user: data.user }); });

// Webhook Stripe app.post('/webhook', bodyParser.raw({ type: 'application/json' }), async (req, res) => { const sig = req.headers['stripe-signature']; let event; try { event = stripe.webhooks.constructEvent(req.body, sig, process.env.STRIPE_WEBHOOK_SECRET); } catch (err) { return res.status(400).send(Webhook Error: ${err.message}); }

const subscription = event.data.object; const email = subscription.metadata.email; const isPremium = ['customer.subscription.updated', 'customer.subscription.created'].includes(event.type);

await supabase.from('users').update({ abonnement: isPremium ? 'premium' : 'gratuit' }).eq('email', email);

res.json({ received: true }); });

app.listen(4242, () => console.log('Serveur backend opérationnel'));

// === 2. SUPABASE EDGE FUNCTION (importPros) === // import.ts import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'; import { createClient } from 'https://esm.sh/@supabase/supabase-js';

serve(async () => { const supabase = createClient(Deno.env.get('SUPABASE_URL'), Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')); const response = await fetch('https://api.annuaire-sante.fr/api/professionnels?specialite=coiffure,esthetique'); const pros = await response.json();

for (const p of pros) { const geo = await fetch(https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(p.adresse)}); const coords = await geo.json(); if (!coords.length) continue; await supabase.from('professionnels').insert({ type: p.specialite, nom: p.nom, adresse: p.adresse, ville: p.ville, telephone: p.telephone, departement: p.departement, latitude: coords[0].lat, longitude: coords[0].lon, }); }

return new Response(JSON.stringify({ ok: true }), { status: 200 }); });

// === 3. FRONTEND React Native (App.jsx) === // - Authentification (connexion, inscription avec validation regex) // - Carte (MapView) avec géolocalisation utilisateurs & pros // - Création de créneaux et prise de RDV // - Échange de créneau (gratuit pour premium, 1€ sinon) // - Abonnement Stripe (Checkout URL) // - Notifications push (Expo Push API) // - Chat en temps réel (Supabase Realtime)

// === 4. ENV (.env) === // SUPABASE_URL=... // SUPABASE_SERVICE_ROLE_KEY=... // STRIPE_SECRET_KEY=... // STRIPE_WEBHOOK_SECRET=...

// === 5. ORGANISATION DU DÉPÔT === // timeswap-app/ // ├── frontend/ (Expo project) // ├── backend/ (Node.js Express) // └── supabase/functions/importPros/

// === PRÊT POUR GITHUB === // Pour publier : crée un dépôt vide, initie git, push le dossier local. // git init && git remote add origin <url> && git push -u origin main

