# Sito web dell'associazione NES

Indirizzo del sito: **https://albertoscarduelli.github.io/NES/** (versione inglese: `/NES/en/`)

Il sito ha due parti ben separate:

| Parte | Dove si trova | Chi la tocca |
|---|---|---|
| **Contenuti**: testi, eventi, notizie, foto, contatti | cartelle `content/` e `data/`, immagini in `static/images/` | chiunque aggiorni il sito |
| **Struttura e grafica**: impaginazione, colori, menu | cartelle `layouts/` e `assets/`, file `hugo.toml` | solo chi sviluppa il sito |

Dopo ogni modifica salvata su GitHub, il sito si aggiorna da solo in 1–2 minuti.

---

## Come aggiornare i contenuti

### Metodo 1 – Pannello di modifica (consigliato)

1. Vai su **https://app.pagescms.org** e premi **Sign in with GitHub**.
2. Scegli il progetto **NES**.
3. A sinistra trovi le voci da modificare:
   - **Dati dell'associazione**: nome, logo, email, telefono, indirizzo, mappa, social
   - **Home – Italiano / English**: copertina, riquadri "Cosa facciamo", invito finale
   - **Pagine**: Chi siamo, Diventa socio, Galleria, Contatti
   - **Eventi** e **Notizie**: aggiungi, modifica o elimina
   - **Galleria fotografica**: carica le foto con le didascalie
4. Premi **Save**. Fatto!

> **Lingue**: ogni contenuto esiste in due versioni, italiana e inglese. Se aggiungi un evento in italiano, ricordati di aggiungerlo anche in inglese.
> Perché il pulsante IT/EN porti dall'uno all'altro, scrivi lo **stesso "Codice di collegamento"** in entrambi (es. `festa-estate-2027`).

### Metodo 2 – Direttamente su GitHub

Apri il file su github.com, premi la matita ✏️, modifica e premi **Commit changes**.

Un evento, per esempio `content/it/eventi/concerto-primavera.md`, è fatto così:

```
---
title: Concerto di primavera
date: 2027-04-17
ora: "21:00"
luogo: Parco comunale
immagine: /images/evento-concerto.svg
description: Una serata di musica dal vivo. Ingresso libero.
---

Qui il testo completo dell'evento. **Grassetto**, *corsivo*, elenchi…
```

Gli eventi con data passata finiscono da soli in "Eventi passati".

### Video

I video si caricano sul **canale YouTube** dell'associazione; nel sito si incolla solo il link
(va bene sia `https://www.youtube.com/watch?v=...` sia `https://youtu.be/...`).

- **In un evento o in una notizia**: campo "Video YouTube" (nel file: riga `video: "link"`).
  Il video compare sotto il testo e la scheda mostra l'etichetta ▶ Video.
- **Nella galleria**: sezione "Video" della Galleria fotografica (file `data/galleria.yaml`).

Per rispetto della privacy (GDPR) YouTube viene contattato solo quando il visitatore preme ▶.

---

## Dove si trova cosa

```
content/it/              pagine in italiano
  _index.md              testo di benvenuto della home
  chi-siamo.md, associati.md, galleria.md, contatti.md
  eventi/                un file per ogni evento
  notizie/               un file per ogni notizia
content/en/              le stesse pagine in inglese
data/associazione.yaml   nome, contatti, social, posizione sulla mappa
data/it/home.yaml        copertina e riquadri della home (italiano)
data/en/home.yaml        copertina e riquadri della home (inglese)
data/galleria.yaml       elenco delle foto della galleria
static/images/           tutte le immagini
i18n/                    piccole scritte fisse dell'interfaccia (es. "Leggi tutto")
hugo.toml                indirizzo del sito e voci del menu
assets/css/style.css     grafica (i colori sono in cima al file)
```

---

## Per lo sviluppatore

Il sito è generato con [Hugo](https://gohugo.io) e pubblicato con GitHub Actions (`.github/workflows/hugo.yml`).

Per vedere l'anteprima sul proprio PC:

```
hugo server
```

e aprire http://localhost:1313/NES/

**Prima pubblicazione** (una sola volta): su GitHub, nel repository, vai in
*Settings → Pages → Build and deployment → Source* e scegli **GitHub Actions**.
