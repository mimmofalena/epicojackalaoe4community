# Documentazione del Progetto – EpicoJackal AoE4 Community

## Indice

1. [Panoramica del Progetto](#1-panoramica-del-progetto)
2. [Architettura](#2-architettura)
   - 2.1 [Stack Tecnologico](#21-stack-tecnologico)
   - 2.2 [Struttura delle Directory](#22-struttura-delle-directory)
   - 2.3 [Architettura Frontend](#23-architettura-frontend)
   - 2.4 [Flusso dei Dati](#24-flusso-dei-dati)
   - 2.5 [Integrazioni Esterne](#25-integrazioni-esterne)
3. [Pattern Utilizzati](#3-pattern-utilizzati)
4. [Funzionalità Principali](#4-funzionalità-principali)
5. [Punti di Forza](#5-punti-di-forza)
6. [Punti di Possibile Miglioramento](#6-punti-di-possibile-miglioramento)
7. [Pattern Consigliati per il Futuro](#7-pattern-consigliati-per-il-futuro)
8. [Variabili d'Ambiente](#8-variabili-dambiente)
9. [Come Avviare il Progetto](#9-come-avviare-il-progetto)

---

## 1. Panoramica del Progetto

**EpicoJackal AoE4 Community** è un portale web per la comunità italiana di *Age of Empires 4* (AoE4), gestita dallo streamer EpicoJackal. Il sito aggrega funzionalità sociali, competitive e di intrattenimento in un'unica piattaforma:

- Classifica nazionale e regionale dei giocatori italiani
- Integrazione con Discord e Twitch
- Gioco a quiz multiplayer in tempo reale (*Beasty Trivia*)
- Calcolatore del *Taffuz Number* (distanza tra giocatori nel grafo delle partite)
- Directory dei coach e calendario eventi

Il progetto è deployato su **Vercel** e non richiede un backend dedicato, sfruttando le API Route di Next.js e servizi terzi.

---

## 2. Architettura

### 2.1 Stack Tecnologico

| Layer | Tecnologia | Versione |
|---|---|---|
| Framework | Next.js (App Router) | 16.1.6 |
| UI Library | React | 19.2.3 |
| Linguaggio | TypeScript | ^5 |
| Styling | Tailwind CSS | ^4 |
| Icone | Lucide React | ^0.577.0 |
| WebSocket (client) | Socket.io Client | ^4.8.3 |
| Analytics | Vercel Analytics + Speed Insights | ^2.0 |
| Deploy | Vercel | – |

### 2.2 Struttura delle Directory

```
epicojackalaoe4community/
├── app/                          # Core applicativo (Next.js App Router)
│   ├── beasty/                   # Feature: Gioco trivia multiplayer
│   │   └── page.tsx
│   ├── components/
│   │   ├── home/                 # Sezioni riutilizzabili della homepage
│   │   │   ├── HeroSection.tsx
│   │   │   ├── DiscordOverviewSection.jsx
│   │   │   ├── FeaturesSection.jsx
│   │   │   ├── TwitchSection.tsx
│   │   │   ├── CoachingSection.tsx
│   │   │   ├── EventsSection.jsx
│   │   │   ├── JoinSection.jsx
│   │   │   └── FooterSection.jsx
│   │   ├── LoadingLink.tsx        # Link con stato di caricamento globale
│   │   ├── NavigationLoaderProvider.tsx
│   │   └── NavigationLoaderReset.tsx
│   ├── config/
│   │   └── site.js               # Configurazione centralizzata (streamer, coach, eventi)
│   ├── data/
│   │   └── leaderboardRegions.json  # Mappa profilo → regione
│   ├── leaderboard/              # Feature: Classifiche
│   │   ├── page.tsx
│   │   ├── LeaderboardClient.tsx
│   │   ├── centro/page.tsx
│   │   ├── nord/page.tsx
│   │   ├── sud/page.tsx
│   │   ├── switzerland/page.tsx
│   │   └── _components/
│   │       └── RegionalLeaderboardPage.tsx
│   ├── lib/                      # Logica di business e integrazioni API
│   │   ├── aoe4world.ts
│   │   ├── regionLeaderboard.ts
│   │   ├── leaderboardRegions.ts
│   │   ├── twitch.ts
│   │   ├── discord.js
│   │   ├── getTwitchLiveMap.ts
│   │   └── beasty/
│   │       ├── types.ts
│   │       ├── useBeasty.ts
│   │       └── socket.ts
│   ├── taffuznumber/             # Feature: Calcolatore Taffuz Number
│   │   └── page.tsx
│   ├── layout.tsx                # Root layout con provider e metadata
│   ├── page.tsx                  # Homepage
│   └── globals.css               # Stili globali e animazioni
├── public/
│   ├── images/                   # Hero images, icone civiltà, badge rank
│   └── audio/                    # Inno svizzero (easter egg)
├── next.config.ts
├── tsconfig.json
├── postcss.config.mjs
├── eslint.config.mjs
└── package.json
```

### 2.3 Architettura Frontend

L'applicazione adotta il modello **ibrido Server/Client Components** di Next.js App Router:

```
┌────────────────────────────────────────────────────────────┐
│                    Next.js App Router                       │
│                                                            │
│  ┌─────────────────────┐    ┌────────────────────────────┐ │
│  │   Server Components  │    │    Client Components       │ │
│  │  (SSR / ISR)         │    │   (CSR, interattivi)       │ │
│  │                      │    │                            │ │
│  │  - app/page.tsx      │    │  - LeaderboardClient.tsx   │ │
│  │  - leaderboard/      │    │  - beasty/page.tsx         │ │
│  │    page.tsx          │    │  - taffuznumber/page.tsx   │ │
│  │  - RegionalLeader-   │    │  - NavigationLoader-       │ │
│  │    boardPage.tsx     │    │    Provider.tsx            │ │
│  └─────────────────────┘    └────────────────────────────┘ │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │               Lib / API Layer                        │  │
│  │   aoe4world.ts  │  twitch.ts  │  discord.js          │  │
│  │   regionLeaderboard.ts  │  leaderboardRegions.ts     │  │
│  └─────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┴──────────────────┐
        │                                    │
   AoE4World API                     Socket.io Server
   Twitch API                        (Beasty Game)
   Discord Widget API
```

**Server Components** gestiscono il fetch iniziale dei dati con caching ISR (Incremental Static Regeneration), garantendo SEO e performance elevate senza JavaScript lato client.

**Client Components** si occupano dell'interattività: ordinamento della classifica, gioco in tempo reale, ricerca del Taffuz Number.

### 2.4 Flusso dei Dati

```
Utente
  │
  ├─[Visita Home]──► Server Component (app/page.tsx)
  │                       │
  │                       ├── fetch Twitch API (revalidate: 60s)
  │                       └── fetch Discord Widget API (revalidate: 60s)
  │
  ├─[Visita Classifica]──► Server Component (leaderboard/page.tsx)
  │                             │
  │                             └── aoe4world.ts → AoE4World API (revalidate: 300s)
  │                                 │
  │                                 └──► LeaderboardClient.tsx (CSR)
  │                                       - Sorting
  │                                       - Filtering
  │                                       - Pagination (URL search params)
  │
  ├─[Taffuz Number]──► Client Component (taffuznumber/page.tsx)
  │                         │
  │                         └── BFS bidirezionale → AoE4World API (on-demand)
  │
  └─[Beasty Trivia]──► Client Component (beasty/page.tsx)
                             │
                             └── useBeasty.ts → Socket.io Server (real-time)
```

### 2.5 Integrazioni Esterne

| Servizio | Utilizzo | File | Caching |
|---|---|---|---|
| **AoE4World API** | Dati classifica, profili giocatori, storico partite | `app/lib/aoe4world.ts` | ISR 5 min |
| **Twitch API** | Stream live, miniature, viewer count | `app/lib/twitch.ts` | ISR 1 min |
| **Discord Widget** | Statistiche server, canali, membri online | `app/lib/discord.js` | ISR 1 min |
| **Socket.io Server** | Gioco trivia in tempo reale | `app/lib/beasty/socket.ts` | Real-time |
| **Vercel Analytics** | Performance e Web Vitals | `app/layout.tsx` | – |
| **Google AdSense** | Monetizzazione | `app/layout.tsx` | – |

---

## 3. Pattern Utilizzati

### 3.1 Server/Client Component Split (Next.js App Router)
I **Server Components** si occupano del data fetching con ISR, mentre i **Client Components** gestiscono lo stato UI interattivo. Questa separazione riduce il JavaScript inviato al browser e migliora i Core Web Vitals.

### 3.2 Repository / Service Layer (API Abstraction)
Le integrazioni con API esterne (`aoe4world.ts`, `twitch.ts`, `discord.js`) sono incapsulate in moduli dedicati nella directory `app/lib/`. Le pagine non comunicano direttamente con le API, ma attraverso queste funzioni di servizio. Questo isola i dettagli di implementazione e facilita il testing e la sostituzione dei provider.

### 3.3 Context API + Custom Hook (State Management)
Il provider `NavigationLoaderProvider` espone uno stato globale di caricamento tramite React Context. Il custom hook `useNavigationLoader()` consente ai componenti figli di accedere allo stato senza prop drilling. Analogamente, `useBeasty()` incapsula tutta la logica Socket.io del gioco trivia, separando la business logic dalla presentazione.

### 3.4 Container/Presenter Pattern
`RegionalLeaderboardPage` è un componente "container" che recupera i dati e li passa a `LeaderboardClient`, componente "presenter" puramente orientato alla UI. Questa separazione migliora la riusabilità e la testabilità.

### 3.5 Configuration Object Pattern
Il file `app/config/site.js` centralizza dati configurativi (streamers, coaches, eventi, features). Modificare questi dati non richiede interventi nel codice dei componenti, riducendo il rischio di regressioni.

### 3.6 Incremental Static Regeneration (ISR)
Le pagine che richiedono dati aggiornati ma non in tempo reale utilizzano `revalidate` per aggiornare la cache in background (AoE4World a 5 minuti, Twitch/Discord a 1 minuto), bilanciando freschezza dei dati e performance.

### 3.7 Bidirectional BFS (Graph Algorithm)
Il calcolatore del Taffuz Number implementa una **ricerca BFS bidirezionale** sul grafo delle sconfitte in partita. La ricerca espande simultaneamente dal nodo sorgente e dal nodo target, riducendo la complessità da O(b^d) a O(b^(d/2)) e gestendo i limiti di rate sulle API con un budget fisso di richieste (`MAX_TOTAL_REQUESTS: 40.000`).

### 3.8 Observer / Event-Driven Architecture (Socket.io)
Il gioco Beasty si basa su un sistema di eventi Socket.io (pattern Observer). Il client reagisce a eventi del server (`game:question`, `game:reveal`, `game:finished`) aggiornando lo stato locale. Questa architettura disaccoppia il server dai client e permette facilmente di aggiungere nuovi tipi di evento.

### 3.9 In-Memory Cache (Player Profile Cache)
In `aoe4world.ts`, una struttura `Map` mantiene in memoria i profili già scaricati durante lo stesso ciclo di rendering server-side, evitando richieste HTTP duplicate per lo stesso giocatore durante l'arricchimento della leaderboard.

---

## 4. Funzionalità Principali

### 4.1 Homepage
Pagina composita con sezioni indipendenti: Hero, Discord overview, Feature highlights, Twitch live, Coaching, Events, Join CTA, Footer. Ogni sezione è un componente React riutilizzabile in `app/components/home/`.

### 4.2 Classifiche
- **Classifica nazionale**: Top 50 giocatori italiani per rating 1v1 (dati da AoE4World API).
- **Classifiche regionali**: Nord, Centro, Sud, Svizzera. I giocatori vengono filtrati tramite una mappa profilo_id → regione definita in `leaderboardRegions.json`.
- Funzionalità: ordinamento per rating (1v1, 2v2, 3v3, 4v4), paginazione, badge rank.

### 4.3 Beasty Trivia
Gioco a quiz multiplayer (WebSocket). Categorie: Civiltà, Unità, Landmark, Edifici, Epoche. Comprende: creazione/join room, sistema di punteggio, moltiplicatori, power-up (doppio punteggio) e schermata risultati finale.

### 4.4 Taffuz Number
Calcola la "distanza" tra un qualsiasi giocatore e "Taffuz" nel grafo delle sconfitte di partite ranked 1v1. Restituisce il percorso più breve con i nomi dei giocatori intermedi e un'etichetta qualitativa ("Direct Slayer", "Elite Hunter", ecc.).

---

## 5. Punti di Forza

### ✅ Architettura Moderna e Performante
L'utilizzo di Next.js App Router con la distinzione Server/Client Components e ISR garantisce performance elevate e ottimi punteggi Core Web Vitals. Il carico computazionale pesante (fetch API, trasformazione dati) viene eseguito lato server.

### ✅ Stack Aggiornato
Il progetto usa le versioni più recenti di React (19), Next.js (16) e Tailwind CSS (4), garantendo accesso alle ultime ottimizzazioni e funzionalità del framework.

### ✅ Type Safety Completa
TypeScript è adottato con strict mode su tutto il codebase. I tipi per le risposte API, gli stati del gioco e le regioni geografiche sono ben definiti, riducendo i bug a runtime.

### ✅ Buona Separazione delle Responsabilità
La distinzione tra layer di servizio (`lib/`), configurazione (`config/`), componenti UI (`components/`) e pagine (`app/*/page.tsx`) rende il codice navigabile e manutenibile.

### ✅ Algoritmo Sofisticato
Il BFS bidirezionale per il Taffuz Number è una soluzione efficiente al problema della distanza nei grafi, documentata con commenti chiari e con gestione dei limiti di rate API.

### ✅ Real-Time ben Gestito
L'hook `useBeasty` incapsula tutta la complessità Socket.io in modo pulito. La gestione degli eventi, dello stato della stanza, del punteggio e del ciclo di vita del gioco è centralizzata e facilmente estendibile.

### ✅ Caching Strategico
Il caching ISR a livelli differenti (1 min per dati in tempo reale come Twitch, 5 min per dati semi-statici come leaderboard) bilancia freschezza e performance in modo consapevole.

### ✅ Deploy Serverless Zero-Config
L'utilizzo di Vercel elimina la necessità di gestire infrastruttura, garantendo scaling automatico e deploy continuo.

---

## 6. Punti di Possibile Miglioramento

### ⚠️ Assenza di Test Automatici
Il progetto non ha nessuna suite di test (né unit test, né integration test, né end-to-end test). Funzioni critiche come il BFS (`taffuznumber`), le trasformazioni dei dati leaderboard e la logica di gioco (`useBeasty`) non sono coperte da test. Qualsiasi modifica rischia di introdurre regressioni non rilevate.

**Suggerimento**: Introdurre Vitest per unit/integration test e Playwright per E2E.

---

### ⚠️ Componenti di Grandi Dimensioni (God Components)
`LeaderboardClient.tsx` (~24 KB), `beasty/page.tsx` (788 righe) e `taffuznumber/page.tsx` (788 righe) sono componenti molto grandi che mescolano UI, logica di stato e logica di business. Questo rende difficile il testing isolato e la manutenzione.

**Suggerimento**: Estrarre la logica di stato in custom hooks dedicati (come già fatto con `useBeasty`) e dividere i componenti in sotto-componenti più piccoli e focalizzati.

---

### ⚠️ Mancanza di Gestione degli Errori Strutturata
Non c'è un sistema di Error Boundary globale in React né una gestione uniforme degli errori API nelle pagine. Se un'API esterna fallisce, il comportamento può essere imprevedibile.

**Suggerimento**: Implementare Error Boundaries React sulle sezioni critiche e una UI di fallback esplicita (es. schermata "dati non disponibili") per ogni feature che dipende da API esterne.

---

### ⚠️ Nessuna Autenticazione/Autorizzazione
Il sito è completamente pubblico. Il gioco Beasty accetta connessioni da chiunque senza autenticazione, rendendo possibile l'abuse (spam di room, bot).

**Suggerimento**: Introdurre autenticazione leggera (es. NextAuth.js con provider Discord) per proteggere le funzionalità multiplayer e identificare i giocatori con i loro account Discord.

---

### ⚠️ Logica di Business nel Layer di Presentazione
In `taffuznumber/page.tsx`, l'intero algoritmo BFS (incluse le chiamate API) risiede nel componente pagina. Questo viola il principio di separazione delle responsabilità e rende il codice difficile da testare.

**Suggerimento**: Estrarre la logica BFS in un modulo separato in `app/lib/taffuznumber.ts` o in un custom hook, lasciando al componente solo la responsabilità della presentazione.

---

### ⚠️ Assenza di Variabili d'Ambiente Documentate
Non esiste un file `.env.example` che documenti le variabili d'ambiente necessarie. Un nuovo sviluppatore non può sapere quali configurare senza leggere l'intero codice sorgente.

**Suggerimento**: Creare un file `.env.example` con tutte le variabili richieste e i relativi commenti esplicativi.

---

### ⚠️ Mix di JavaScript e TypeScript
Alcuni componenti e moduli usano `.jsx`/`.js` (`DiscordOverviewSection.jsx`, `FeaturesSection.jsx`, `discord.js`, `site.js`) invece di TypeScript. Questo introduce zone senza type safety.

**Suggerimento**: Migrare progressivamente i file `.js`/`.jsx` a `.ts`/`.tsx` per uniformità e sicurezza dei tipi.

---

### ⚠️ Nessun Sistema di Logging
Non è presente alcun sistema di logging lato server. In produzione, gli errori nelle chiamate API o nella logica server-side non vengono tracciati strutturalmente.

**Suggerimento**: Integrare un provider di logging (es. Sentry per il monitoraggio degli errori, o il logging nativo di Vercel) per avere visibilità sugli errori in produzione.

---

### ⚠️ Dati Regionali Hard-Coded nel JSON
La mappa `leaderboardRegions.json` richiede aggiornamento manuale ogni volta che un giocatore cambia regione o entra nella community. Non c'è un meccanismo di auto-discovery o un pannello di amministrazione.

**Suggerimento**: Considerare l'utilizzo di un CMS headless (es. Contentful, Sanity) o di un database semplice (es. Vercel KV, PlanetScale) per gestire questi dati in modo dinamico.

---

## 7. Pattern Consigliati per il Futuro

### 🔵 Testing con Vitest + React Testing Library
Adottare **Vitest** (compatibile con il tooling Vite/Turbopack di Next.js) per unit test e **React Testing Library** per i component test. Prioritizzare:
- Unit test per l'algoritmo BFS (`taffuznumber`)
- Test della logica di trasformazione dati in `aoe4world.ts`
- Component test per `LeaderboardClient`

```
npm install -D vitest @testing-library/react @testing-library/user-event @vitejs/plugin-react
```

---

### 🔵 Feature-Based Folder Structure
Con la crescita del progetto, organizzare il codice per **feature** invece che per tipo di file:

```
app/
├── features/
│   ├── leaderboard/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── lib/
│   │   └── types.ts
│   ├── beasty/
│   └── taffuznumber/
├── shared/
│   ├── components/
│   ├── hooks/
│   └── lib/
```

Questo aumenta la coesione intra-feature e riduce l'accoppiamento inter-feature.

---

### 🔵 React Query (TanStack Query) per Data Fetching Client-Side
Per le chiamate API lato client (es. Taffuz Number), sostituire `useState` + `fetch` manuale con **TanStack Query** per ottenere:
- Caching automatico
- Gestione stati loading/error/success
- Retry automatico
- Deduplicazione richieste

```
npm install @tanstack/react-query
```

---

### 🔵 Zod per Validazione degli Schema API
Le risposte delle API esterne (AoE4World, Twitch) vengono attualmente consumate con semplice casting TypeScript. Usare **Zod** per validare gli schema a runtime:

```typescript
import { z } from "zod";

const PlayerSchema = z.object({
  profile_id: z.number(),
  name: z.string(),
  rank: z.number().nullable(),
  rating: z.number().nullable(),
});
```

Questo previene crash silenziosi quando un'API cambia il formato della risposta.

---

### 🔵 NextAuth.js per Autenticazione
Integrare **NextAuth.js** con il provider Discord per autenticare gli utenti. Questo permetterebbe di:
- Associare i profili Discord ai profili AoE4World
- Proteggere il gioco Beasty da abusi
- Personalizzare l'esperienza in base all'utente

```
npm install next-auth
```

---

### 🔵 Storybook per Component Documentation
Introdurre **Storybook** per documentare i componenti UI in isolamento. Utile per:
- Sviluppare componenti senza dover avviare l'intera applicazione
- Documentazione visiva della design system
- Test visivo delle varianti dei componenti (rank badges, stati della leaderboard)

---

### 🔵 Struttura a Micro-Feature con Colocation dei Test
Per ogni feature, collocare i test accanto al codice che testano:

```
app/features/leaderboard/
├── LeaderboardClient.tsx
├── LeaderboardClient.test.tsx    ← test collocati
├── hooks/
│   └── useLeaderboardSort.ts
│   └── useLeaderboardSort.test.ts
```

Questo migliora la manutenibilità e garantisce che i test vengano aggiornati insieme al codice.

---

## 8. Variabili d'Ambiente

Creare un file `.env.local` nella root del progetto con le seguenti variabili:

```env
# Twitch API (https://dev.twitch.tv/console)
TWITCH_CLIENT_ID=<il_tuo_client_id>
TWITCH_CLIENT_SECRET=<il_tuo_client_secret>

# Socket.io Server URL per il gioco Beasty
NEXT_PUBLIC_REALTIME_URL=<url_del_server_socketio>
```

> **Nota**: Le variabili con prefisso `NEXT_PUBLIC_` sono esposte al browser. Non inserire segreti in queste variabili.

---

## 9. Come Avviare il Progetto

### Prerequisiti
- Node.js >= 18
- npm >= 9

### Installazione

```bash
# 1. Clonare il repository
git clone https://github.com/Mimmofalena/epicojackalaoe4community.git
cd epicojackalaoe4community

# 2. Installare le dipendenze
npm install

# 3. Configurare le variabili d'ambiente
cp .env.example .env.local
# Editare .env.local con i propri valori

# 4. Avviare il server di sviluppo
npm run dev
```

Aprire [http://localhost:3000](http://localhost:3000) nel browser.

### Script Disponibili

| Comando | Descrizione |
|---|---|
| `npm run dev` | Avvia il server di sviluppo con hot reload |
| `npm run build` | Crea la build di produzione |
| `npm run start` | Avvia il server di produzione (richiede build) |
| `npm run lint` | Esegue ESLint sul codebase |
