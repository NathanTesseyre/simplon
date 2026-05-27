---
title: "Composants d'un frontend moderne"
tags:
  - frontend
  - composants
  - SPA
  - SSR
  - React
  - Vue
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche_frontend]]
  - [[fiche-deploiement-frontend]]
  - [[fiche_api]]
---

# 📘 Composants d'un frontend moderne

## Vue d'ensemble

Un frontend est la partie d'une application que l'utilisateur voit et avec laquelle il interagit directement dans son navigateur. Il se compose de plusieurs couches qui travaillent ensemble.

```
┌─────────────────────────────────────────────┐
│              Navigateur                     │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │           Couche UI                  │   │
│  │  Composants visuels (HTML/CSS)       │   │
│  ├─────────────────────────────────────┤   │
│  │         Couche logique               │   │
│  │  État, routage, validation           │   │
│  ├─────────────────────────────────────┤   │
│  │         Couche données               │   │
│  │  Appels API, cache, synchronisation  │   │
│  └─────────────────────────────────────┘   │
│                   │                         │
└───────────────────│─────────────────────────┘
                    │ HTTP/WebSocket
                    ▼
              Backend / API
```

## 1. Stratégies de rendu

Le choix du mode de rendu est fondamental — il impacte les performances, le SEO, et la complexité du déploiement.

### CSR — Client-Side Rendering (SPA)

Le serveur envoie une page HTML presque vide + du JavaScript. Le navigateur télécharge le JS et construit le DOM lui-même.

```
Serveur → <html><body><div id="root"></div><script src="app.js"></script></body></html>
              ↓ (le navigateur charge et exécute app.js)
Navigateur construit toute l'interface en JS
```

**Frameworks :** React (CRA/Vite), Vue, Angular, Svelte

**Avantages :** navigation fluide sans rechargement de page, expérience riche, séparation totale front/back

**Inconvénients :** premier chargement lent (le JS doit être chargé avant que l'utilisateur voit quoi que ce soit), mauvais SEO par défaut (les robots ne voient pas le contenu rendu en JS)

**Cas d'usage :** tableaux de bord, apps SaaS, outils internes, apps qui nécessitent une authentification

### SSR — Server-Side Rendering

Le serveur génère le HTML complet pour chaque requête et l'envoie au navigateur.

```
Requête → Serveur Node.js → génère le HTML avec les données → envoie au navigateur
         ← HTML complet avec le contenu déjà là
```

**Frameworks :** Next.js (React), Nuxt (Vue), SvelteKit, Remix

**Avantages :** bon SEO (le contenu est dans le HTML), affichage rapide du premier écran

**Inconvénients :** charge serveur plus élevée, temps de réponse dépend du serveur

**Cas d'usage :** e-commerce, blogs, sites marketing, tout ce qui doit être indexé par Google

### SSG — Static Site Generation

Le HTML est généré **au moment du build**, pas à chaque requête. Les fichiers statiques sont ensuite servis depuis un CDN.

```
npm run build → génère tous les fichiers HTML/CSS/JS → déploiement sur CDN
Requête →  CDN (serveur le plus proche de l'utilisateur) → fichier HTML statique
```

**Frameworks :** Next.js (`getStaticProps`), Astro, Hugo, Gatsby

**Avantages :** ultra-rapide (fichiers pré-générés), très sécurisé (pas de serveur dynamique), cheap à héberger

**Inconvénients :** contenu figé jusqu'au prochain build, pas adapté aux données très dynamiques

**Cas d'usage :** documentation, blogs, landing pages, sites marketing

### ISR — Incremental Static Regeneration (Next.js)

Hybride SSG + SSR : les pages sont statiques mais peuvent être régénérées en arrière-plan après un délai.

```javascript
// Next.js : cette page est statique, régénérée toutes les 60 secondes
export async function getStaticProps() {
  const data = await fetch('https://api.example.com/posts');
  return {
    props: { posts: await data.json() },
    revalidate: 60  // secondes
  };
}
```

## 2. Gestion de l'état

L'état (state) représente les données qui changent dans le temps et qui font évoluer l'interface.

### État local — dans un composant

```javascript
// React avec useState
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);  // état local
  return (
    <button onClick={() => setCount(count + 1)}>
      Cliqué {count} fois
    </button>
  );
}
```

### État global — partagé entre composants

Quand plusieurs composants distants ont besoin des mêmes données (ex: utilisateur connecté, thème, panier).

**Solutions courantes :**
- **React Context** — solution native, suffisante pour des cas simples
- **Zustand** — léger, intuitif, recommandé pour les nouveaux projets React
- **Redux Toolkit** — plus structuré, adapté aux grosses applications
- **Pinia** (Vue) — standard pour Vue 3

```javascript
// Zustand — état global simple
import { create } from 'zustand';

const useAuthStore = create((set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null }),
}));

// Dans n'importe quel composant
function Header() {
  const { user, logout } = useAuthStore();
  return user ? <button onClick={logout}>Déconnexion ({user.name})</button> : null;
}
```

### Données serveur — React Query / TanStack Query

Pour les données qui viennent d'une API, React Query gère le cache, le refetch, et les états de chargement/erreur automatiquement.

```javascript
import { useQuery } from '@tanstack/react-query';

function UsersList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(r => r.json()),
    staleTime: 5 * 60 * 1000,  // données fraîches pendant 5 minutes
  });

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

## 3. Routage

Le routeur gère la navigation entre les pages sans rechargement complet.

```javascript
// React Router v6
import { BrowserRouter, Routes, Route, Link, useParams } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Accueil</Link>
        <Link to="/users">Utilisateurs</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users" element={<UsersList />} />
        <Route path="/users/:id" element={<UserDetail />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

function UserDetail() {
  const { id } = useParams();  // récupère le paramètre :id de l'URL
  // ...
}
```

## 4. Appels API

```javascript
// Fetch natif
async function getUsers() {
  const response = await fetch('/api/users', {
    method: 'GET',
    headers: { 'Authorization': `Bearer ${token}` }
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

// Avec Axios (plus de fonctionnalités, meilleure gestion des erreurs)
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
});

// Intercepteur pour ajouter le token à chaque requête
api.interceptors.request.use(config => {
  config.headers.Authorization = `Bearer ${localStorage.getItem('token')}`;
  return config;
});

const users = await api.get('/users');
await api.post('/users', { name: 'Alice', email: 'alice@example.com' });
```

## 5. Performance

```html
<!-- Lazy loading des images -->
<img src="photo.jpg" loading="lazy" alt="..." />

<!-- Preload des ressources critiques -->
<link rel="preload" href="fonts/ma-police.woff2" as="font" crossorigin />
```

```javascript
// Code splitting — charger un composant seulement quand nécessaire
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));  // chargé à la demande

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Dashboard />
    </Suspense>
  );
}
```

**Métriques clés (Core Web Vitals) :**
- **LCP** (Largest Contentful Paint) : temps avant que le plus grand élément visible soit affiché. Objectif < 2.5s
- **CLS** (Cumulative Layout Shift) : stabilité visuelle (éviter les éléments qui sautent). Objectif < 0.1
- **FID/INP** : réactivité aux interactions utilisateur. Objectif < 200ms

Outils de mesure : Lighthouse (Chrome DevTools), PageSpeed Insights, WebPageTest.

## 6. Sécurité côté navigateur

```javascript
// Ne JAMAIS stocker de données sensibles dans localStorage
// localStorage est accessible par tout JS de la page (vulnérable XSS)
localStorage.setItem('token', jwt);  // ⚠️ risqué pour des données sensibles

// Préférer les cookies httpOnly pour les tokens d'authentification
// (inaccessibles au JavaScript, donc protégés contre le XSS)
// → géré côté serveur via Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict

// Toujours échapper le contenu utilisateur
// React le fait automatiquement dans JSX
// Mais dangerouslySetInnerHTML contourne cette protection :
<div dangerouslySetInnerHTML={{ __html: userContent }} />  // ⚠️ risque XSS
```

**Content Security Policy (CSP)** — empêche l'exécution de scripts non autorisés :
```
Content-Security-Policy: default-src 'self'; script-src 'self' cdn.example.com
```

## 7. Tests frontend

```javascript
// Test unitaire d'un composant avec Vitest + Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import Counter from './Counter';

describe('Counter', () => {
  it('incrémente le compteur au clic', () => {
    render(<Counter />);
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Cliqué 0 fois');
    fireEvent.click(button);
    expect(button).toHaveTextContent('Cliqué 1 fois');
  });
});

// Test end-to-end avec Playwright
import { test, expect } from '@playwright/test';

test('login flow', async ({ page }) => {
  await page.goto('http://localhost:3000/login');
  await page.fill('[name=email]', 'user@example.com');
  await page.fill('[name=password]', 'secret');
  await page.click('button[type=submit]');
  await expect(page).toHaveURL('/dashboard');
});
```

---

### ✅ À retenir

- **CSR/SPA** = expérience riche, mauvais SEO natif → apps avec auth
- **SSR** = bon SEO, rendu côté serveur → e-commerce, sites publics
- **SSG** = ultra-rapide, statique → docs, blogs, sites marketing
- **État local** (`useState`) pour un composant, **état global** (Zustand/Pinia) pour toute l'app
- **React Query** = gestion des données serveur avec cache intégré
- Ne pas stocker de secrets dans le localStorage

**Lire ensuite →** [[fiche-deploiement-frontend]] pour la mise en production.
