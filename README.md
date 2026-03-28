# EpicoJackal AoE4 Community

Portale della comunità italiana di **Age of Empires 4** gestita da EpicoJackal.

## 📖 Documentazione

Per la documentazione completa del progetto (architettura, pattern utilizzati, punti di forza e miglioramenti consigliati) consultare:

👉 **[DOCUMENTATION.md](./DOCUMENTATION.md)**

## 🚀 Avvio Rapido

```bash
# Installare le dipendenze
npm install

# Configurare le variabili d'ambiente
cp .env.example .env.local

# Avviare il server di sviluppo
npm run dev
```

Aprire [http://localhost:3000](http://localhost:3000) nel browser.

## Script Disponibili

| Comando | Descrizione |
|---|---|
| `npm run dev` | Server di sviluppo con hot reload |
| `npm run build` | Build di produzione |
| `npm run start` | Server di produzione |
| `npm run lint` | Lint del codice |

## Deploy

Il progetto è deployato su [Vercel](https://vercel.com). Ogni push sul branch principale attiva un deploy automatico.
