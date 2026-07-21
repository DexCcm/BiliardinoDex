# 🏓 Bigliardino DEX

Dashboard di un torneo di **calcio balilla (biliardino)** interno a **Dataexpert**.
Classifiche giocatori e team, statistiche, sfida Blu vs Rosso, registrazione partite —
il tutto in **una singola pagina HTML**, senza build né framework, con dati sincronizzati
in tempo reale via **Firebase Realtime Database**.

![Bigliardino DEX](https://img.shields.io/badge/stack-HTML%20%2B%20CSS%20%2B%20JS%20vanilla-f2b721) ![Firebase](https://img.shields.io/badge/Firebase-Realtime%20Database-ffca28)

---

## ✨ Funzionalità

- **Riepiloghi generali** — KPI (partite, giocatori, gol totali, media/gara)
- **Blu vs Rosso** — vittorie, barra %, gol fatti / subiti / differenza reti
- **🏆 Classifica giocatori** e **👥 Classifica team** — con medaglie 🥇🥈🥉 ai primi 3 e Win% a scala di colore
- **🐐 Team più forte / Team sfortunato**
- **⚽ Gol fatti** e **🧤 Gol subiti** per coppia
- **Schede giocatori** — win%, forma (ultime 5), ruolo (difesa/attacco/jolly), miglior partner
- **😭 Valle di lacrime** — i "cappotti" (partite chiuse a zero)
- **🎲 Estrattore di match** — pagina dedicata che sorteggia la prossima partita tra i presenti, con un sorteggio quasi tutto casuale ma che favorisce leggermente le coppie che hanno giocato meno insieme (animazioni a tema)
- **Registrazione partite** con validazioni e scelta giocatori da menu a tendina (solo censiti)
- **👥 Gestione giocatori** — aggiunta e ricolorazione senza toccare la Console Firebase
- **Sync live multi-utente** — ogni modifica appare in tempo reale su tutti i dispositivi
- **Intro animata** e layout responsive (desktop / tablet / mobile)

---

## 🚀 Come usarlo

Apri `index.html` in un browser (doppio click, oppure servilo con qualsiasi hosting statico:
GitHub Pages, Netlify, ecc.). Non serve alcun build.

> I dati vivono su Firebase Realtime Database; `localStorage` fa da cache offline e da
> fallback se Firebase non è raggiungibile.

---

## 🔧 Configurazione Firebase

La configurazione è nel blocco `<script type="module">` in fondo a `index.html`.
Per il **Realtime Database** è obbligatorio il campo `databaseURL` (non presente nella config
di default della Console):

```js
const firebaseConfig = {
  apiKey: "…",
  authDomain: "biliardinodex.firebaseapp.com",
  databaseURL: "https://biliardinodex-default-rtdb.europe-west1.firebasedatabase.app", // regione europe-west1
  projectId: "biliardinodex",
  // …
};
```

### Popolamento iniziale dei dati

Due possibilità:

1. **Automatico** — al primo avvio, se il database è vuoto, l'app carica da sola il seed
   (giocatori + 56 partite del torneo).
2. **Manuale** — importa `firebase-import.json` da Console → Realtime Database → nodo radice →
   menu ⋮ → **Importa JSON**.
   ⚠️ L'import alla radice **sostituisce l'intero database**: fallo solo all'inizio.

### Regole di sicurezza

Per un uso interno rapido bastano regole aperte:

```json
{ "rules": { ".read": true, ".write": true } }
```

> ⚠️ **Attenzione:** con regole aperte **chiunque conosca l'URL del database può leggere e
> scrivere**. Le API key web di Firebase non sono segrete (sono pensate per stare nel client),
> ma se pubblichi questo repository su GitHub l'URL diventa facilmente reperibile.
> Per un uso non strettamente privato, introduci l'**autenticazione** e regole più restrittive.

---

## 🗂️ Struttura del database

```
biliardinodex
├── players/   → anagrafica giocatori   (key = nome)      { name, color, tint }
└── matches/   → partite                (key = push-id)   { blue:[n1,n2], bs, red:[n1,n2], rs }
```

- `blue` = coppia a **sinistra**, `red` = coppia a **destra**
- `bs` / `rs` = punteggio blu / rosso
- Tutte le statistiche (classifiche, coppie, cappotti, KPI) sono **derivate** dalle partite

---

## 📁 File del progetto

| File | Contenuto |
|------|-----------|
| `index.html` | L'applicazione completa (HTML + CSS + JS + Firebase) |
| `firebase-import.json` | Dati iniziali (giocatori + partite) pronti da importare |
| `SPECIFICHE.md` | Documentazione tecnica completa / handoff |
| `risultati.txt` | Risultati grezzi del torneo (fonte originale) |
| `firebase.txt` | Snippet di configurazione Firebase |

---

## 🛠️ Stack

HTML, CSS e JavaScript vanilla in singolo file · Firebase Realtime Database (SDK modulare v11 via CDN) ·
font Google Fredoka + Nunito. Nessuna dipendenza da installare, nessun processo di build.

---

_Torneo Dataexpert · sinistra = Blu, destra = Rosso_
