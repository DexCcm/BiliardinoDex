# Bigliardino DEX — Specifiche complete del progetto

> Documento di handoff per riprendere il lavoro su qualsiasi dispositivo.
> Le sessioni di Claude Code sono locali: per continuare altrove, apri il progetto e fornisci questo file come contesto.

---

## 1. Cos'è

Dashboard di un torneo di **calcio balilla (biliardino)** interno a **Dataexpert**.
Applicazione **single-file**: tutto (HTML + CSS + JS) è in un unico file autonomo, senza dipendenze esterne
tranne i font Google (Fredoka + Nunito). Nessun build, nessun framework.

- **File principale:** `Bigliardino-DEX/index.html`
- **Come aprirlo:** doppio click sul file, si apre nel browser. Funziona anche offline (i font richiedono connessione ma degradano con fallback di sistema).
- **Percorso attuale:** `C:\Users\Utente\Desktop\WABI\3d - Terrarium\Bigliardino-DEX\index.html`

### Origine
Ricostruito da un progetto **claude.ai/design** (framework proprietario "DC" con tag `sc-if`/`sc-for`/`{{ }}`),
convertito in HTML/CSS/JS vanilla.
- Design project id: `0a97e9a7-fe61-493b-b874-b58b3997506d`
- File sorgente design: `Bigliardino DEX.html` / `Bigliardino Dex.dc.html` / `Logo Dataexpert.dc.html`

---

## 2. Design system

**Font**
- Display / titoli: **Fredoka** (400–700)
- Testo: **Nunito** (400–900)

**Palette (CSS variables in `:root`)**
| Nome | Valore | Uso |
|------|--------|-----|
| `--bg` | `#f4efe4` | sfondo pagina (crema) |
| `--card` | `#fffdf8` | card/pannelli |
| `--ink` / `--ink2` | `#3b3c46` / `#43434e` | testo scuro |
| `--muted` / `--muted2` | `#9a95a5` / `#a29dac` | testo secondario |
| `--blue` / `--blue-soft` | `#3f6fb5` / `#6f9fe0` | Squadra Blu |
| `--red` / `--red-soft` | `#cf5b5b` / `#ec8f8f` | Squadra Rossa |
| `--green` | `#6bbf7b` | vittorie / positivo |
| `--green-field` | `#9fd6a3` | campo da gioco |
| `--gold` / `--gold2` | `#f2b721` / `#e6b23f` | accenti dataexpert, pallina |
| `--purple` | `#b48ce0` | accento |

**Convenzione partite:** sinistra = **Blu**, destra = **Rosso**.

**Animazioni CSS principali** (`@keyframes`): `rise`, `pop`, `bob`, `barGrow`, `pass` (pallina che rimbalza tra gli omini),
`cameraZoom` + `skyOut` + `logoReveal` (intro), `leafGrow` (foglioline logo), `twinkle`, `ringSpin`.
Rispetta `prefers-reduced-motion` (animazioni disattivate e intro saltata).

---

## 3. Struttura della pagina

**Intro animata** (4,6s, con pulsante "Salta"): zoom su un campo da biliardino stilizzato + reveal del logo dataexpert.
Poi appare la **dashboard**.

**Header (sticky):**
- Icona bigliardino 3D (SVG inline)
- Titolo: **Bigliardino data​expert** — "data" scuro, "expert" oro (`#f2b90d`)
- Sottotitolo: **TORNEO · 16 GIU 2026**
- Pulsanti: **＋ Nuova partita** (apre la modale) · **↺ Rivedi intro**

**Sezioni della dashboard (in quest'ordine di importanza — NON cambiare senza richiesta):**
1. **Riepiloghi generali** — Hero con KPI (Partite, Giocatori, Gol totali, Media/gara)
2. **Blu vs Rosso** — vittorie, barra %, gol fatti/subiti/differenza
3. **🏆 Classifica giocatori** + **👥 Classifica coppie** (affiancate)
4. **Coppia più forte / Coppia sfortunata** (migliore–peggiore)
5. **⚽ Gol fatti** (per coppia)
6. **🧤 Gol subiti** (per coppia)
7. **Schede giocatori** (win%, vinte, perse, forma, ruolo, partner)
8. **Ultime partite**
9. **😭 Valle di lacrime** — i "cappotti" (partite chiuse con 0 gol subiti dall'avversario)
10. **Tutti i risultati** — tabella completa (con eliminazione al passaggio del mouse)

**Favicon:** mini campo da biliardino (SVG data-URI nell'`<head>`, `<link rel="icon" id="favicon">`).

---

## 4. Modello dati & persistenza

**localStorage key:** `bigliardino_dex_matches_v1`

**Formato:** array di oggetti partita
```js
{ blue: ["Nome1", "Nome2"], bs: 10, red: ["Nome3", "Nome4"], rs: 6 }
// bs = punteggio blu, rs = punteggio rosso
```

- Al primo avvio, se localStorage è vuoto, viene caricato il **seed** di 56 partite del torneo (costante `SEED`, formato riga `[b1,b2,bs,r1,r2,rs]`, convertito con `rowToObj`).
- Funzioni chiave: `loadMatches()` / `saveMatches(arr)`.
- Ogni modifica ricalcola tutto tramite `compute(objs)` e ri-renderizza (`render()`).

**Giocatori:** dedotti dalle partite. Colori noti in `colorMap`/`tintMap`; per nuovi nomi c'è un fallback deterministico (`colorFor`/`tintFor` con hash su palette `FALLBACK`).

---

## 5. Funzionalità

- **Registrare una partita** (pulsante ＋ Nuova partita → modale):
  - 2 giocatori Blu + 2 Rossi (input con `datalist` → autocompletamento dei nomi esistenti, ma si possono aggiungere **nuovi giocatori** digitando)
  - Punteggi con stepper +/−
  - **Validazioni:** 4 giocatori tutti diversi · nessun pareggio · punteggi ≥ 0
  - Al salvataggio: persiste, ricalcola, toast di conferma (riconosce i cappotti), scroll + flash sulla partita nuova
- **Eliminare una partita:** hover sulla riga in "Tutti i risultati" → pulsante ×, con conferma
- **Gestire i giocatori** (pulsante 👥 Giocatori in header → modale):
  - Lista di tutti i giocatori (visti nelle partite ∪ anagrafica), ognuno con selettore colore
  - Cambio colore → salvato su Firebase (nodo `players`) e applicato live a tutte le sezioni; la tinta chiara è derivata in automatico dal colore
  - Aggiunta nuovo giocatore (nome + colore); i nuovi nomi diventano subito selezionabili nel form partita
  - I giocatori NON si eliminano dalla UI (scelta voluta: sono derivati dalle partite)
- **Intro:** si vede al primo caricamento; "Rivedi intro" la ripete; saltabile

---

## 6. Statistiche calcolate (funzione `compute`)

Per **giocatore**: partite giocate, vinte, perse, win%, gol fatti/subiti, differenza reti, ruolo (Difesa/Attacco/Jolly in base a quante volte gioca in prima/seconda posizione), forma (ultime 5), miglior partner.
Per **coppia** (min. 3 partite insieme): win%, gol fatti, gol subiti, classifica, migliore/peggiore.
Aggregati **Blu vs Rosso**, KPI generali, ultime partite, cappotti ("valle di lacrime").

---

## 7. Campo da biliardino (dettaglio importante)

Disposizione **reale** delle stecche, alternate tra le squadre: **1–2–5–3 per lato**
(portiere 1, difesa 2, centrocampo 5, attacco 3). Rosso speculare.
Sequenza da sinistra a destra (8 stecche): `1B · 2B · 3R · 5B · 5R · 3B · 2R · 1R`.
Definita nell'array `rods` (dentro `compute`) e in `introRodsSpec` (intro) — devono restare coerenti.

---

## 8. Responsive

- **Desktop** ≥ 900px: layout a griglia multi-colonna.
- ≤ 900px: hero e sezioni a colonna singola, campo sopra.
- ≤ 680px: header compatto (pulsanti solo icona, sottotitolo nascosto), card 1 colonna, modale full-width.
- ≤ 400px: colonne classifica ridotte.
- Nessun overflow orizzontale (verificato a 375px e 1280px).

---

## 9. Persistenza — Firebase Realtime DB (FATTO)

La persistenza è ora su **Firebase Realtime Database** con sincronizzazione live multi-utente.
localStorage resta come **cache offline** (primo paint veloce + fallback se Firebase non è raggiungibile).

**Struttura del database (nodi radice):**
```
biliardinodex
├── players/   → anagrafica giocatori, key = nome:  { name, color, tint }
└── matches/   → partite, key = push-id:  { blue:[n1,n2], bs, red:[n1,n2], rs }
```
- **`players`** è la fonte dei colori/stili: `colorMap`/`tintMap` in `index.html` restano solo come DEFAULT, il nodo `players` li sovrascrive/estende. Basta aggiungere un giocatore nel DB per stilizzarlo in tutte le sezioni.
- **`matches`**: sinistra = blu, destra = rosso; push-key cronologiche.

**Come è implementato (in fondo a `index.html`, `<script type="module">`):**
- SDK modulare Firebase v11 via CDN ESM (`firebase-app.js` + `firebase-database.js`).
- Config con `databaseURL` regione **europe-west1** (obbligatorio per RTDB, non era nel `firebase.txt`).
- `onValue(rootRef)` legge insieme `players` e `matches`: applica prima l'anagrafica (`window.applyRemotePlayers` → merge su colorMap/tintMap), poi le partite (`window.applyRemoteMatches` → array con `_key`) → `render()`. Ogni client vede gli aggiornamenti in tempo reale.
- Scritture partite: `window.dbAdd(rec)` (= `push` su `matches`) e `window.dbRemove(key)` (= `remove`). Il codice principale li usa se presenti, altrimenti fallback localStorage.
- **Seed automatico:** al primissimo avvio, se `matches` è vuoto, carica il `SEED` di 56 partite; se anche `players` è vuoto, carica l'anagrafica di default (`window.getPlayersSeed`).
- `compute()` trasporta `_key` (7° elemento di `M`); il delete usa la push-key invece dell'indice.

**Import iniziale (manuale):** il file `firebase-import.json` (in cartella) contiene `players` + `matches` già pronti. Console → Realtime Database → nodo radice → menu ⋮ → **Importa JSON** (⚠️ l'import alla radice sostituisce l'intero DB).

**⚠️ Regole di sicurezza RTDB:** il DB deve permettere lettura/scrittura. Per un uso interno rapido, in Firebase Console → Realtime Database → Regole:
```json
{ "rules": { ".read": true, ".write": true } }
```
(Regole aperte = chiunque con l'URL può leggere/scrivere. Per irrigidire in seguito serve introdurre l'autenticazione.)

---

## 10. Come riprendere il lavoro (prompt suggerito)

> "Sto continuando il progetto **Bigliardino DEX**. Il file è `Bigliardino-DEX/index.html`, single-file HTML/CSS/JS vanilla, dati in localStorage (`bigliardino_dex_matches_v1`). Leggi `Bigliardino-DEX/SPECIFICHE.md` per il contesto completo. Voglio: [descrivere la modifica]."

### Cronologia modifiche già fatte
- Conversione da design DC a single-file vanilla, con intro, tutte le sezioni, animazioni, mobile.
- Aggiunta registrazione/eliminazione partite + localStorage + validazioni.
- Riordino sezioni per importanza (vedi §3).
- Pulsante "Nuova partita" solo in header (rimosso il FAB).
- Correzione disposizione stecche campo a 1–2–5–3.
- Header: icona bigliardino + "Bigliardino **dataexpert**" (DEX sostituito dal logo dataexpert) + "TORNEO · 16 GIU 2026".
- Favicon con mini campo da biliardino.
- Migrazione persistenza da localStorage a **Firebase Realtime Database** con sync live (vedi §9); localStorage come cache/fallback; seed automatico se DB vuoto; delete tramite push-key.
- Giocatori promossi a entità strutturata nel nodo `players` (name/color/tint); `colorMap`/`tintMap` restano come default.
- Aggiunta UI **👥 Giocatori** in header per aggiungere/ristilizzare giocatori senza passare dalla Console Firebase (nessuna eliminazione).
- Win% nelle classifiche (giocatori e coppie) colorata con scala pastello rosso→oro→verde (`pctColor`).
- Classifiche giocatori/team: rimosse le righe alternate; medaglie 🥇🥈🥉 accanto ai primi 3; intestazioni colonne aggiunte anche alla Classifica team; 🐐 (GOAT) sul Team più forte.
