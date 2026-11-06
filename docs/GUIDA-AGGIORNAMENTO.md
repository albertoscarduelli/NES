# Sito NES – Guida all'aggiornamento

Questa guida spiega, passo per passo, come **aggiornare i contenuti** del sito: eventi, notizie, foto, video, testi delle pagine e dati dell'associazione.
Non serve nessuna competenza tecnica per i metodi 1 e 2.

Per modifiche alla grafica o alla struttura vedi invece [ARCHITETTURA.md](ARCHITETTURA.md).

---

## Indice

1. [Come funziona l'aggiornamento](#1-come-funziona-laggiornamento)
2. [Primo accesso e permessi](#2-primo-accesso-e-permessi)
3. [Metodo 1 – Pannello di modifica (Pages CMS)](#3-metodo-1--pannello-di-modifica-pages-cms)
4. [Operazioni passo per passo](#4-operazioni-passo-per-passo)
5. [Lingue: italiano e inglese](#5-lingue-italiano-e-inglese)
6. [Immagini: come prepararle](#6-immagini-come-prepararle)
7. [Video](#7-video)
8. [Scrivere i testi (formattazione)](#8-scrivere-i-testi-formattazione)
9. [Metodo 2 – Modifica diretta su GitHub](#9-metodo-2--modifica-diretta-su-github)
10. [Metodo 3 – Dal PC con git e anteprima locale](#10-metodo-3--dal-pc-con-git-e-anteprima-locale)
11. [Controllare che la pubblicazione sia andata a buon fine](#11-controllare-che-la-pubblicazione-sia-andata-a-buon-fine)
12. [Se qualcosa va storto](#12-se-qualcosa-va-storto)
13. [Promemoria rapido](#13-promemoria-rapido)

---

## 1. Come funziona l'aggiornamento

```
  Tu modifichi un contenuto          GitHub salva la modifica        Dopo 1–2 minuti
  (pannello, GitHub o PC)   ───▶     e ricostruisce il sito   ───▶   il sito online è aggiornato
```

- I contenuti sono **file** conservati nel repository GitHub `albertoscarduelli/NES`.
- Ogni salvataggio diventa una "versione" (commit): **nulla va perso**, si può sempre tornare indietro.
- Non bisogna "caricare" il sito da nessuna parte: la pubblicazione è automatica.
- Il sito si ricostruisce anche **ogni notte**, così gli eventi già passati si spostano da soli in "Eventi passati".

Tre modi per aggiornare, dal più semplice al più tecnico:

| Metodo | Per chi | Serve |
|---|---|---|
| **1. Pannello Pages CMS** | tutti | un browser e un account GitHub |
| **2. Modifica su github.com** | chi vuole cambiare al volo un testo | un browser e un account GitHub |
| **3. Dal PC con git** | chi modifica anche grafica o struttura | git, Hugo, un editor |

---

## 2. Primo accesso e permessi

### 2.1 Account GitHub

Chi aggiorna il sito deve avere un **account GitHub** (gratuito): https://github.com/signup

### 2.2 Dare il permesso a un'altra persona (lo fa il proprietario del repository)

1. Vai su https://github.com/albertoscarduelli/NES
2. **Settings** → **Collaborators** → **Add people**.
3. Scrivi il nome utente o l'email GitHub della persona → **Add**.
4. La persona riceve un'email di invito e deve **accettarlo**.

Per togliere il permesso: stessa pagina, pulsante **Remove** accanto al nome.

> Chi è collaboratore può modificare **tutto** il repository. Dai l'accesso solo a persone di fiducia; in ogni caso ogni modifica resta registrata con il nome di chi l'ha fatta e può essere annullata.

### 2.3 Collegare Pages CMS (solo la prima volta)

1. Vai su **https://app.pagescms.org** e premi **Sign in with GitHub**.
2. GitHub chiede di autorizzare Pages CMS: conferma.
3. Se il progetto **NES** non compare nell'elenco, Pages CMS propone di installare la sua "app" GitHub: scegli **Only select repositories** → **NES** → **Install**.
4. Apri il progetto **NES** (ramo `main`).

Per gli accessi successivi basta il punto 1.

---

## 3. Metodo 1 – Pannello di modifica (Pages CMS)

Una volta aperto il progetto, a sinistra trovi il menu con queste voci:

| Voce | Cosa modifica |
|---|---|
| **Dati dell'associazione** | nome, logo, descrizione breve (IT/EN), email, telefono, indirizzo, coordinate della mappa, social |
| **Home – Italiano** / **Home – English** | copertina (titolo, sottotitolo, immagine, pulsanti), riquadri "Cosa facciamo", invito finale |
| **Pagine – Italiano** / **Pages – English** | Home (testo di benvenuto), Chi siamo, Diventa socio, Galleria, Contatti |
| **Eventi – Italiano** / **Events – English** | gli eventi |
| **Notizie – Italiano** / **News – English** | le notizie |
| **Galleria fotografica** | foto con didascalie e video della galleria |
| **Media** | tutte le immagini caricate |

Regole generali:
- Dopo ogni modifica premi **Save**: la modifica viene salvata su GitHub e il sito si aggiorna in 1–2 minuti.
- I campi con l'asterisco sono obbligatori.
- Se un campo facoltativo resta vuoto, l'elemento corrispondente semplicemente non compare sul sito (es. un social senza link non mostra l'icona).

> L'aspetto esatto dei pulsanti può cambiare leggermente con gli aggiornamenti di Pages CMS; i nomi delle voci di menu e dei campi sono invece quelli definiti per questo sito.

---

## 4. Operazioni passo per passo

### 4.1 Aggiungere un evento

1. **Eventi – Italiano** → **Add an entry** (o "New").
2. Compila:

   | Campo | Esempio | Note |
   |---|---|---|
   | Titolo | Festa d'estate | obbligatorio |
   | Data | 21/06/2027 | obbligatorio: è la data dell'evento |
   | Orario | 18:00 – 23:00 | testo libero |
   | Luogo | Parco comunale | testo libero |
   | Immagine | *carica una foto* | vedi §6; se manca viene usata un'immagine standard |
   | Breve descrizione | Musica, giochi e cena insieme | 1–2 frasi: compare nelle schede |
   | Video YouTube | https://youtu.be/… | facoltativo, vedi §7 |
   | Codice di collegamento IT/EN | festa-estate-2027 | vedi §5 |
   | Testo completo | programma, dettagli, come partecipare | |

3. **Save**.
4. Ripeti in **Events – English** con i testi in inglese e **lo stesso codice di collegamento**.

L'evento compare in home tra i "Prossimi eventi" (i primi 3 in ordine di data) e nella pagina Eventi. Il giorno dopo la data, passa da solo in "Eventi passati".

### 4.2 Modificare o cancellare un evento

- **Modificare:** apri l'evento dall'elenco, cambia i campi, **Save**.
- **Cancellare:** apri l'evento → menu "⋯" / **Delete**. Ricordati di cancellare anche la versione nell'altra lingua.

> Gli eventi passati di solito **non si cancellano**: restano in "Eventi passati" come archivio delle attività dell'associazione.

### 4.3 Aggiungere una notizia

Come per gli eventi, in **Notizie – Italiano** e poi in **News – English**.
Campi: Titolo, Data (di pubblicazione), Immagine, Breve descrizione, Video YouTube, Codice di collegamento, Testo completo.
In home compaiono le 3 notizie più recenti; nella pagina Notizie tutte, 9 per pagina.

### 4.4 Modificare i dati dell'associazione

**Dati dell'associazione** → modifica → **Save**. Le modifiche valgono in tutto il sito (intestazione, piè di pagina, pagina Contatti).

Per la **mappa** servono latitudine e longitudine della sede:
1. Apri https://www.openstreetmap.org e cerca l'indirizzo.
2. Tasto destro sul punto esatto → **Mostra indirizzo** (o "Show address").
3. A sinistra compaiono due numeri, per esempio `45.46420, 9.19000`: il primo è la **latitudine**, il secondo la **longitudine**.
4. Usa il **punto** come separatore decimale, non la virgola.

Per i **social**: incolla l'indirizzo completo della pagina (es. `https://www.facebook.com/nomepagina`). Lascia vuoto per nascondere l'icona.

### 4.5 Modificare la home

**Home – Italiano** (e poi **Home – English**):
- **Copertina**: titolo grande, sottotitolo, immagine di sfondo (consigliata orizzontale, almeno 1600 px di larghezza), due pulsanti.
  - Nel campo "pagina" dei pulsanti scrivi l'indirizzo della pagina **senza** `/NES/` e senza lingua: `chi-siamo/`, `eventi/`, `associati/`, `contatti/` (in inglese: `about/`, `events/`, `join/`, `contact/`).
- **Riquadri "Cosa facciamo"**: si possono aggiungere, togliere e riordinare; per ognuno scegli icona, titolo e testo. Consigliati 3 (o 6) riquadri, per riempire bene le righe.
- **Invito finale**: titolo, testo e pulsante del riquadro verde in fondo alla home.

Il **testo di benvenuto** (sotto la copertina) si modifica invece in **Pagine → Home**.

### 4.6 Modificare una pagina (Chi siamo, Diventa socio…)

**Pagine – Italiano** → scegli la pagina → modifica **Titolo**, **Sottotitolo** e **Testo** → **Save**. Poi la versione inglese in **Pages – English**.

Nelle pagine **Galleria** e **Contatti** il testo è facoltativo: foto e contatti vengono presi da "Galleria fotografica" e "Dati dell'associazione".

> Le voci del **menu** non si modificano dal pannello: se serve aggiungere o togliere una pagina dal menu, è una modifica tecnica (vedi ARCHITETTURA.md, §17).

### 4.7 Aggiungere foto alla galleria

1. **Galleria fotografica** → sezione **Foto** → **Add** (aggiungi elemento).
2. **Foto**: carica l'immagine (vedi §6).
3. **Didascalia (Italiano)** e **Caption (English)**.
4. **Save**.

L'ordine sul sito è quello dell'elenco nel pannello: le foto in cima all'elenco compaiono per prime. Per togliere una foto dalla galleria eliminala dall'elenco (il file resta comunque in **Media**).

---

## 5. Lingue: italiano e inglese

- Ogni contenuto esiste in **due copie separate**, una per lingua. Il sito non traduce automaticamente.
- Se un contenuto esiste solo in italiano, la versione inglese del sito semplicemente non lo mostra.
- Il pulsante **IT / EN** in alto porta alla stessa pagina nell'altra lingua **solo se le due copie sono collegate**.

**Come si collegano un evento o una notizia nelle due lingue:** scrivi lo stesso **"Codice di collegamento IT/EN"** in entrambe le copie.
Usa un codice breve, senza spazi né accenti, che non hai già usato, per esempio `festa-estate-2027`, `assemblea-2027`, `notizia-raccolta-fondi`.

Se il codice manca o è diverso, tutto funziona lo stesso, ma il pulsante EN/IT porta alla home dell'altra lingua invece che all'evento tradotto.

Le **date** sono scritte automaticamente nel formato giusto per ogni lingua ("17 aprile 2027" / "17 April 2027"): basta inserirle una volta nel campo Data.

---

## 6. Immagini: come prepararle

| | Consiglio |
|---|---|
| Formato | **JPG** per le foto, **PNG** o **SVG** per loghi e grafiche |
| Dimensioni | lato lungo **1600–2000 pixel** al massimo (la copertina della home almeno 1600 px di larghezza) |
| Peso | possibilmente **sotto i 500 KB** per foto: un sito leggero si apre più veloce sui telefoni |
| Proporzioni | eventi e notizie: **orizzontali** (16:10 o 3:2); le immagini vengono ritagliate automaticamente nelle schede, quindi tieni il soggetto al centro |
| Nome del file | minuscole, senza spazi né accenti: `festa-estate-2027.jpg` (non `IMG 2034 Festa d'Estate.JPG`) |

Per ridurre una foto scattata con il telefono (spesso 4–8 MB), puoi usare gratuitamente **https://squoosh.app** (Google, funziona nel browser e non carica le foto su server): trascina la foto, imposta "Resize" a 1600 di larghezza e qualità 75–80, scarica.

**Privacy:** prima di pubblicare foto in cui si riconoscono persone, assicurati di avere il loro consenso (per i **minori** serve quello dei genitori).

---

## 7. Video

I video **non si caricano sul sito**, ma sul **canale YouTube** dell'associazione; nel sito si incolla solo il link.

1. Carica il video su YouTube (può essere "Pubblico" o "Non in elenco"; **non** "Privato", altrimenti nessuno lo vede).
2. Su YouTube premi **Condividi** e copia il link (es. `https://youtu.be/AbCdEfGhIjK`).
3. Incollalo:
   - in un **evento** o in una **notizia**, nel campo **Video YouTube**: il video compare sotto il testo e la scheda mostra l'etichetta **▶ Video**;
   - nella **galleria**: **Galleria fotografica** → sezione **Video** → **Add** → link + titolo italiano + titolo inglese.

Vanno bene tutti i formati di link YouTube: `youtube.com/watch?v=…`, `youtu.be/…`, `youtube.com/shorts/…`.

Per rispetto della privacy, il video viene caricato da YouTube **solo quando il visitatore preme ▶**.

> Nel sito di esempio ci sono video con link segnaposto (`XXXXXXXXXXX`): vanno sostituiti con video veri o cancellati.

---

## 8. Scrivere i testi (formattazione)

Nei campi "Testo" del pannello c'è un editor con i pulsanti per **grassetto**, *corsivo*, titoli, elenchi e link, come in un programma di videoscrittura.

Consigli:
- Usa i **titoli** dell'editor (H2, H3) per dividere i testi lunghi in sezioni; non usare il grassetto al posto dei titoli.
- Non serve scrivere il titolo della pagina nel testo: viene già mostrato in alto.
- Per un link a un'altra pagina del sito usa l'indirizzo completo (es. `https://albertoscarduelli.github.io/NES/contatti/`).
- Frasi brevi e paragrafi corti: sul telefono si leggono meglio.

Se modifichi i file direttamente (metodi 2 e 3), il testo è in **Markdown**:

| Scrivi | Ottieni |
|---|---|
| `**testo**` | **testo** in grassetto |
| `*testo*` | *testo* in corsivo |
| `## Titolo` | un titolo di sezione |
| `- voce` (una per riga) | elenco puntato |
| `1. voce` | elenco numerato |
| `[testo del link](https://…)` | link |
| `![descrizione](/images/foto.jpg)` | immagine nel testo |
| `> testo` | riquadro di citazione |
| riga vuota | nuovo paragrafo |

---

## 9. Metodo 2 – Modifica diretta su GitHub

Utile per correggere al volo un testo.

1. Vai su https://github.com/albertoscarduelli/NES e apri il file (es. `content/it/eventi/concerto-primavera.md`).
2. Premi la **matita ✏️** ("Edit this file").
3. Modifica. Attenzione alla parte tra le due righe `---` in cima: rispetta il formato `campo: valore` e metti tra virgolette i valori con i due punti (`ora: "20:30"`).
4. Premi **Commit changes…**, scrivi una breve descrizione (es. "Corretto orario concerto") → **Commit changes**.

**Creare un nuovo evento da GitHub:** apri la cartella `content/it/eventi/` → **Add file** → **Create new file** → nome `nome-evento.md` → copia la struttura di un evento esistente.

**Caricare un'immagine da GitHub:** apri la cartella `static/images/` → **Add file** → **Upload files**. Nel campo immagine scriverai poi `/images/nome-file.jpg`.

Dove si trovano i contenuti:

| Contenuto | File |
|---|---|
| Eventi | `content/it/eventi/` e `content/en/eventi/` |
| Notizie | `content/it/notizie/` e `content/en/notizie/` |
| Pagine | `content/it/*.md` e `content/en/*.md` |
| Testo di benvenuto della home | `content/it/_index.md`, `content/en/_index.md` |
| Copertina, riquadri e invito della home | `data/it/home.yaml`, `data/en/home.yaml` |
| Dati dell'associazione | `data/associazione.yaml` |
| Galleria (foto e video) | `data/galleria.yaml` |
| Immagini | `static/images/` |

---

## 10. Metodo 3 – Dal PC con git e anteprima locale

Per chi ha il repository sul proprio computer (cartella `E:\Alberto\SVIL\SitiWeb\NES`) con **git** e **Hugo** installati.

```bash
cd E:/Alberto/SVIL/SitiWeb/NES
git pull                       # 1. SEMPRE per primo: scarica le modifiche fatte da altri
hugo server                    # 2. anteprima su http://localhost:1313/NES/
```

3. Modifica i file con un editor (consigliato VS Code). L'anteprima nel browser si aggiorna da sola a ogni salvataggio.
4. Quando è tutto a posto, interrompi l'anteprima con **Ctrl+C** e invia:

```bash
git add -A
git commit -m "Aggiunto evento Festa d'estate"
git push
```

Note:
- Se `git push` viene rifiutato con un messaggio che contiene "fetch first", qualcuno ha modificato il sito nel frattempo: esegui `git pull` e poi di nuovo `git push`.
- In anteprima locale la mappa e i video potrebbero non caricarsi in alcuni browser integrati: online funzionano.
- L'anteprima locale mostra anche gli eventi futuri e passati esattamente come saranno online.

---

## 11. Controllare che la pubblicazione sia andata a buon fine

1. Vai su https://github.com/albertoscarduelli/NES → scheda **Actions**.
2. In cima c'è l'ultima esecuzione di **"Pubblica il sito"**:
   - 🟡 in corso — aspetta un minuto;
   - ✅ verde — pubblicato: ricarica il sito (se non vedi la modifica, **Ctrl+F5** o svuota la cache del browser);
   - ❌ rossa — **errore**: il sito online **resta alla versione precedente** (non si rompe), vedi §12.

---

## 12. Se qualcosa va storto

### La pubblicazione è fallita (❌ rossa)

Quasi sempre è un errore di formato in un file modificato a mano (metodi 2 e 3):
- un campo nella parte tra `---` senza virgolette e con i due punti (`ora: 20:30` → `ora: "20:30"`);
- una data scritta male (formato corretto: `2027-06-21`);
- un rientro sbagliato in un file `.yaml` (vanno usati gli spazi, e allineati come le righe vicine).

Apri l'esecuzione fallita → **build** → passo **"Genera il sito"**: il messaggio di errore indica il file e la riga. Correggi il file e salva di nuovo.

### Ho fatto una modifica sbagliata e voglio tornare indietro

- **Da GitHub:** apri il file → **History** (cronologia) → scegli la versione buona → copia il testo → modifica il file e incolla → **Commit changes**.
- Se non sei sicuro, chiedi a chi gestisce la parte tecnica: ogni versione precedente è conservata e recuperabile.

### Un evento passato è ancora tra i "prossimi"

Il sito si ricostruisce ogni notte. Se dopo un giorno l'evento è ancora lì: **Actions** → **Pubblica il sito** → **Run workflow**. Se in Actions compare un avviso di workflow disattivato (succede dopo 60 giorni senza modifiche al sito), premi il pulsante per riattivarlo.

### Il pulsante IT/EN non porta all'evento tradotto

Controlla che le due versioni abbiano **lo stesso codice di collegamento** (§5).

### Non vedo la modifica sul sito

- Controlla in **Actions** che la pubblicazione sia terminata (✅).
- Ricarica con **Ctrl+F5**; sul telefono chiudi e riapri la pagina.

---

## 13. Promemoria rapido

| Voglio… | Dove (Pages CMS) |
|---|---|
| aggiungere un evento | Eventi – Italiano **+** Events – English (stesso codice di collegamento) |
| aggiungere una notizia | Notizie – Italiano **+** News – English |
| aggiungere foto | Galleria fotografica → Foto |
| aggiungere un video | campo "Video YouTube" dell'evento/notizia, oppure Galleria fotografica → Video |
| cambiare email, telefono, indirizzo, social | Dati dell'associazione |
| cambiare copertina o riquadri della home | Home – Italiano / Home – English |
| cambiare il testo di benvenuto | Pagine → Home |
| cambiare "Chi siamo" o "Diventa socio" | Pagine – Italiano / Pages – English |
| controllare la pubblicazione | github.com → NES → Actions |
| rendere il sito visibile su Google | modifica tecnica: `bozza = false` in `hugo.toml` |
