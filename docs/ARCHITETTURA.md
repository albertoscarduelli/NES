# Sito NES – Documentazione tecnica

Documento per chi deve **modificare il sito a mano**: struttura, funzionamento interno, ricette per le modifiche più comuni.
È scritto pensando a chi programma (per esempio in C++) ma non ha esperienza di sviluppo web.

Per **aggiornare i contenuti** (eventi, notizie, foto…) non serve niente di tutto questo: vedi [GUIDA-AGGIORNAMENTO.md](GUIDA-AGGIORNAMENTO.md).

---

## Indice

1. [Il quadro generale](#1-il-quadro-generale)
2. [Corrispondenze con il mondo C++](#2-corrispondenze-con-il-mondo-c)
3. [Albero delle cartelle](#3-albero-delle-cartelle)
4. [Il processo di build](#4-il-processo-di-build)
5. [Contenuti: file Markdown e front matter](#5-contenuti-file-markdown-e-front-matter)
6. [Dati: la cartella `data/`](#6-dati-la-cartella-data)
7. [Template: la cartella `layouts/`](#7-template-la-cartella-layouts)
8. [Il linguaggio dei template (Go template) in 10 minuti](#8-il-linguaggio-dei-template-go-template-in-10-minuti)
9. [Multilingua](#9-multilingua)
10. [Indirizzi, percorsi e `baseURL`](#10-indirizzi-percorsi-e-baseurl)
11. [Eventi: prossimi e passati](#11-eventi-prossimi-e-passati)
12. [Video YouTube](#12-video-youtube)
13. [CSS e JavaScript](#13-css-e-javascript)
14. [Pannello di modifica Pages CMS](#14-pannello-di-modifica-pages-cms)
15. [Pubblicazione: GitHub Actions e GitHub Pages](#15-pubblicazione-github-actions-e-github-pages)
16. [Lavorare in locale](#16-lavorare-in-locale)
17. [Ricette: modifiche comuni](#17-ricette-modifiche-comuni)
18. [Debug e problemi frequenti](#18-debug-e-problemi-frequenti)
19. [Riferimenti](#19-riferimenti)

---

## 1. Il quadro generale

Il sito è un **sito statico**: è fatto solo di file HTML, CSS, JavaScript e immagini già pronti. Non c'è un server applicativo, non c'è un database, non c'è codice che gira quando un visitatore apre una pagina.

```
                 sorgenti                         build                        output
 ┌──────────────────────────────────┐      ┌───────────────┐      ┌───────────────────────────┐
 │ content/  (testi .md)            │      │               │      │ public/                   │
 │ data/     (dati .yaml)           │ ───▶ │     Hugo      │ ───▶ │   index.html              │
 │ layouts/  (template .html)       │      │  (generatore) │      │   eventi/…/index.html     │
 │ assets/   (CSS, JS)              │      │               │      │   en/…/index.html         │
 │ static/   (immagini)             │      └───────────────┘      │   css/style.min.<hash>.css│
 │ i18n/     (stringhe fisse)       │                             │   images/…                │
 │ hugo.toml (configurazione)       │                             └─────────────┬─────────────┘
 └──────────────────────────────────┘                                           │
                                                                    GitHub Pages lo serve
                                                                    così com'è ai visitatori
```

- **Hugo** è il generatore: legge i sorgenti e scrive la cartella `public/` con tutte le pagine finali.
- La build avviene **su GitHub** (GitHub Actions) a ogni modifica sul ramo `main`; il risultato viene pubblicato su **GitHub Pages**.
- **Pages CMS** è solo un'interfaccia web che modifica i file del repository al posto dell'utente (fa dei commit). Non è un server del sito.

Conseguenza importante: **tutto ciò che è "dinamico" viene calcolato al momento della build**, non quando il visitatore apre la pagina. Per esempio la divisione tra eventi futuri e passati (vedi §11).

---

## 2. Corrispondenze con il mondo C++

| Nel sito | Analogia C++ / ambiente desktop |
|---|---|
| Hugo | compilatore + sistema di build (come MSBuild) |
| `hugo.toml` | file di progetto / proprietà di build (`.vcxproj`, `.props`) |
| `content/**/*.md` | dati di input; il "front matter" è una `struct` con i campi della pagina |
| `data/*.yaml` | file di configurazione / tabelle di costanti caricati a build-time |
| `layouts/` | il codice che genera l'output (template) |
| `layouts/partials/*.html` | funzioni riutilizzabili (anche con valore di ritorno) |
| `baseof.html` + `{{ block "main" }}` | classe base con metodo virtuale sovrascritto dalle classi derivate (pattern *template method*) |
| `i18n/*.yaml` | tabelle di stringhe localizzate (come i cataloghi di messaggi) |
| `assets/` | sorgenti che subiscono una trasformazione (minificazione, hash) |
| `static/` | risorse copiate così come sono nell'output |
| `public/` | cartella di output della build (come `Bin64\`); **non va nel repository** |
| `hugo server` | build incrementale in "watch mode" + ricarica automatica del browser |
| GitHub Actions (`.github/workflows/hugo.yml`) | pipeline di CI/CD |
| variabili CSS (`--c-primary`) | `constexpr` globali usate in tutto il foglio di stile |
| commit su `main` | "rilascio": parte la pipeline e il sito online si aggiorna |

---

## 3. Albero delle cartelle

```
NES/
├── hugo.toml                   Configurazione: URL, lingue, menu, permalink, parametri
├── .pages.yml                  Configurazione del pannello Pages CMS (moduli di modifica)
├── .gitignore                  File esclusi dal repository (public/, resources/_gen/, .claude/)
├── README.md                   Panoramica breve
├── docs/
│   ├── ARCHITETTURA.md         Questo documento
│   └── GUIDA-AGGIORNAMENTO.md  Guida operativa per aggiornare i contenuti
│
├── content/                    ── CONTENUTI (una cartella per lingua) ──
│   ├── it/
│   │   ├── _index.md           Home: titolo + testo di benvenuto
│   │   ├── chi-siamo.md        Pagina semplice
│   │   ├── associati.md        Pagina semplice
│   │   ├── galleria.md         Solo titolo/descrizione; le foto sono in data/galleria.yaml
│   │   ├── contatti.md         Solo titolo/descrizione; i contatti sono in data/associazione.yaml
│   │   ├── eventi/*.md         Un file per evento   (sezione "eventi")
│   │   └── notizie/*.md        Un file per notizia  (sezione "notizie")
│   └── en/                     Stessa struttura in inglese (about.md, join.md, gallery.md, contact.md…)
│
├── data/                       ── DATI CONDIVISI ──
│   ├── associazione.yaml       Nome, logo, contatti, coordinate mappa, social (indipendenti dalla lingua)
│   ├── galleria.yaml           Foto (con didascalie IT/EN) e video della galleria
│   ├── it/home.yaml            Blocchi della home in italiano (copertina, riquadri, invito)
│   └── en/home.yaml            Blocchi della home in inglese
│
├── i18n/
│   ├── it.yaml                 Stringhe fisse dell'interfaccia in italiano ("Leggi tutto"…)
│   └── en.yaml                 Le stesse in inglese
│
├── layouts/                    ── TEMPLATE (struttura HTML) ──
│   ├── index.html              Home page
│   ├── _default/
│   │   ├── baseof.html         Scheletro comune a tutte le pagine (<html>, <head>, header, footer)
│   │   ├── single.html         Pagina semplice (Chi siamo, Diventa socio, Galleria, Contatti)
│   │   ├── list.html           Elenco generico (fallback per sezioni senza template proprio)
│   │   └── _markup/
│   │       └── render-image.html  "Hook": come vengono resi i ![img](…) nel Markdown
│   ├── eventi/
│   │   ├── list.html           Elenco eventi (prossimi + passati)
│   │   └── single.html         Pagina del singolo evento
│   ├── notizie/
│   │   ├── list.html           Elenco notizie, con paginazione
│   │   └── single.html         Pagina della singola notizia
│   └── partials/               Pezzi riutilizzabili
│       ├── head.html           Contenuto di <head>: titolo, meta, CSS, hreflang
│       ├── header.html         Barra superiore: logo, menu, pulsante ☰
│       ├── lang-switch.html    Pulsanti IT / EN
│       ├── footer.html         Piè di pagina
│       ├── page-header.html    Fascia colorata con titolo delle pagine interne
│       ├── page-title.html     (funzione) restituisce il titolo della pagina
│       ├── event-card.html     Scheda di un evento
│       ├── news-card.html      Scheda di una notizia
│       ├── gallery.html        Griglia foto + lightbox + sezione video
│       ├── contact.html        Schede contatto + mappa OpenStreetMap
│       ├── youtube.html        Lettore YouTube "privacy-first"
│       ├── icon.html           Libreria di icone SVG
│       ├── url.html            (funzione) URL di una risorsa (immagini) con prefisso /NES/
│       └── link.html           (funzione) URL di una pagina nella lingua corrente
│
├── assets/                     ── RISORSE ELABORATE DA HUGO ──
│   ├── css/style.css           Tutto lo stile del sito
│   └── js/main.js              Menu mobile, ombra header, lightbox, avvio video
│
├── static/                     ── RISORSE COPIATE COSÌ COME SONO ──
│   └── images/                 Logo, copertina, immagini di eventi, notizie e galleria
│
└── .github/workflows/hugo.yml  Pipeline di build e pubblicazione
```

Cartelle **generate**, da non modificare e non committare: `public/`, `resources/_gen/`, il file `.hugo_build.lock`.

---

## 4. Il processo di build

Hugo, a ogni build:

1. legge `hugo.toml` e, per **ogni lingua** configurata, costruisce un "sito" separato (IT e EN);
2. carica tutti i file di `content/<lingua>/` come **pagine**, con il loro front matter;
3. carica `data/` e `i18n/`;
4. per ogni pagina sceglie il **template** adatto (§7), lo esegue e scrive l'HTML in `public/`;
5. elabora le risorse di `assets/` richieste dai template (minifica il CSS, aggiunge un hash al nome del file);
6. copia `static/` in `public/` senza modifiche.

Tipi di pagina ("kind") che Hugo crea:

| Kind | Esempio | Template usato |
|---|---|---|
| `home` | `/NES/`, `/NES/en/` | `layouts/index.html` |
| `section` | `/NES/eventi/`, `/NES/notizie/` | `layouts/<sezione>/list.html`, altrimenti `_default/list.html` |
| `page` | `/NES/eventi/concerto-primavera/`, `/NES/chi-siamo/` | `layouts/<sezione>/single.html`, altrimenti `_default/single.html` |

La **sezione** di una pagina è la prima cartella sotto `content/<lingua>/`: `eventi`, `notizie`, oppure nessuna (pagine in radice).
Le tassonomie (tag/categorie) sono disattivate (`disableKinds` in `hugo.toml`).

I tempi di build sono bassissimi (meno di mezzo secondo), quindi non c'è bisogno di ottimizzare nulla.

---

## 5. Contenuti: file Markdown e front matter

Ogni file `.md` ha due parti:

```markdown
---                                  ← inizio front matter (YAML)
title: Concerto di primavera         ← campi della pagina
date: 2027-04-17
ora: "21:00"
luogo: Parco comunale
immagine: /images/evento-concerto.svg
description: Una serata di musica dal vivo.
video: "https://youtu.be/XXXXXXXXXXX"
translationKey: concerto-2027
---                                  ← fine front matter

Testo in **Markdown**: grassetto, *corsivo*, [link](https://…),   ← corpo (.Content)
elenchi, tabelle, ecc.
```

### Campi usati dal sito

| Campo | Dove | Significato | Accesso nei template |
|---|---|---|---|
| `title` | tutti | titolo della pagina | `.Title` |
| `description` | tutti | sottotitolo / testo delle schede / meta description | `.Description` |
| `date` | eventi, notizie | **data dell'evento** o della notizia (`AAAA-MM-GG`) | `.Date` |
| `translationKey` | tutti | lega tra loro le traduzioni (§9) | usato da Hugo |
| `ora` | eventi | orario, testo libero | `.Params.ora` |
| `luogo` | eventi | luogo, testo libero | `.Params.luogo` |
| `immagine` | eventi, notizie | immagine di copertina (percorso sotto `static/`) | `.Params.immagine` |
| `video` | eventi, notizie | link YouTube | `.Params.video` |

I campi standard di Hugo (`title`, `date`, `description`…) si leggono direttamente (`.Title`); quelli personalizzati stanno in `.Params` (`.Params.luogo`). **I nomi dei campi nel front matter sono case-insensitive** per Hugo, ma conviene scriverli sempre in minuscolo.

### Regole YAML da ricordare

- L'indentazione conta (spazi, **mai tab**).
- Metti tra virgolette i valori che contengono `:` o `#` o che iniziano con caratteri speciali: `ora: "20:30"`.
- Le date nel formato `2027-04-17`.

### Nome del file = indirizzo

`content/it/eventi/concerto-primavera.md` → `/NES/eventi/concerto-primavera/`.
Usa solo minuscole, numeri e trattini; niente spazi né accenti.
In inglese l'indirizzo degli eventi e delle notizie si basa sul **titolo** (vedi `permalinks` in §10).

---

## 6. Dati: la cartella `data/`

I file YAML in `data/` sono disponibili in tutti i template come `hugo.Data.<nome file>`; le sottocartelle diventano livelli annidati:

| File | Nei template |
|---|---|
| `data/associazione.yaml` | `hugo.Data.associazione` → `.nome`, `.email`, `.social.facebook`… |
| `data/galleria.yaml` | `hugo.Data.galleria.foto`, `hugo.Data.galleria.video` |
| `data/it/home.yaml` | `hugo.Data.it.home` (letto come `index hugo.Data site.Language.Lang "home"`) |

Convenzione per i campi bilingue nei file **non** divisi per lingua: suffisso `_it` / `_en` (`descrizione_it`, `didascalia_en`, `titolo_it`).
Il template sceglie quello giusto componendo il nome: `index $a (printf "descrizione_%s" site.Language.Lang)`.

> Nota: in Hugo ≥ 0.156 si usa `hugo.Data`; il vecchio `site.Data` / `.Site.Data` è deprecato.

---

## 7. Template: la cartella `layouts/`

### Scheletro comune: `baseof.html`

```html
<html lang="{{ site.Language.Locale }}">
<head>{{ partial "head.html" . }}</head>
<body>
  {{ partial "header.html" . }}
  <main id="main">
    {{ block "main" . }}{{ end }}      ← "metodo virtuale"
  </main>
  {{ partial "footer.html" . }}
  <script src="…main.js" defer></script>
</body>
</html>
```

Ogni template di pagina (`index.html`, `single.html`, `eventi/list.html`…) contiene solo:

```html
{{ define "main" }}
  … HTML specifico della pagina …
{{ end }}
```

che viene inserito al posto di `{{ block "main" . }}`.

### Scelta del template (lookup)

Hugo cerca il template dal più specifico al più generico, come la risoluzione di un overload:

- pagina `content/it/eventi/x.md` → `layouts/eventi/single.html` → `layouts/_default/single.html`
- sezione `/eventi/` → `layouts/eventi/list.html` → `layouts/_default/list.html`
- home → `layouts/index.html`

Per creare un comportamento diverso per una nuova sezione basta creare `layouts/<nuova-sezione>/single.html` e/o `list.html`.

### Pagine semplici con comportamenti extra

`_default/single.html` mostra titolo e testo; in più aggiunge la galleria o i contatti **in base al `translationKey`** della pagina:

```html
{{ if eq .Params.translationKey "gallery" }}{{ partial "gallery.html" . }}{{ end }}
{{ if eq .Params.translationKey "contact" }}{{ partial "contact.html" . }}{{ end }}
```

Per questo nelle pagine Galleria e Contatti il `translationKey` (`gallery`, `contact`) **non va cambiato**.

### La home: `layouts/index.html`

Blocchi, nell'ordine; ognuno compare solo se il relativo dato esiste (`{{ with … }}`):

| # | Blocco | Fonte dati |
|---|---|---|
| 1 | Copertina (`.hero`) | `data/<lingua>/home.yaml` → `hero` |
| 2 | Testo di benvenuto | corpo di `content/<lingua>/_index.md` |
| 3 | Riquadri "Cosa facciamo" | `home.yaml` → `punti_forza` |
| 4 | Prossimi 3 eventi | pagine della sezione `eventi` con data ≥ ieri |
| 5 | Ultime 3 notizie | pagine della sezione `notizie`, le più recenti |
| 6 | Invito finale (`.cta`) | `home.yaml` → `invito` |

Per **spostare** un blocco si spostano le righe corrispondenti in `index.html`; per **toglierlo** si cancella il blocco (o si svuota il dato).

### Partial "funzione"

Alcuni partial non producono HTML ma **restituiscono un valore** con `return`:

```html
{{- $out := … -}}
{{- return $out -}}
```

e si usano come espressioni: `<img src="{{ partial "url.html" .Params.immagine }}">`.
Sono: `url.html`, `link.html`, `page-title.html`. Come in C++, `return` interrompe l'esecuzione del partial; qui si usa un solo `return` finale per leggibilità (e compatibilità con le versioni vecchie di Hugo, che ne ammettevano uno solo).

---

## 8. Il linguaggio dei template (Go template) in 10 minuti

Tutto ciò che sta tra `{{ … }}` è codice; il resto è HTML copiato così com'è.

### Il "punto" `.` è il contesto corrente

Pensalo come un `this` che **cambia** dentro `with` e `range`:

```html
<h1>{{ .Title }}</h1>                 <!-- . = la pagina -->
{{ with .Params.luogo }}
  <p>{{ . }}</p>                      <!-- qui . = il valore di luogo -->
{{ end }}
{{ range .Pages }}
  <li>{{ .Title }}</li>               <!-- qui . = ogni singola pagina dell'elenco -->
{{ end }}
```

`$` è sempre il contesto iniziale del template (la pagina), anche dentro `with`/`range`: `{{ $.Title }}`.

### Costrutti principali

| Costrutto | Significato | Equivalente C++ (circa) |
|---|---|---|
| `{{ $x := valore }}` | dichiara una variabile | `auto x = valore;` |
| `{{ $x = valore }}` | riassegna | `x = valore;` |
| `{{ if cond }}…{{ else if c2 }}…{{ else }}…{{ end }}` | condizione | `if / else` |
| `{{ with v }}…{{ else }}…{{ end }}` | esegue se `v` non è vuoto, e imposta `.` = `v` | `if (auto& p = v; !empty(p))` |
| `{{ range lista }}…{{ end }}` | ciclo; `.` = elemento | `for (auto& e : lista)` |
| `{{ range $k, $v := mappa }}` | ciclo su chiave/valore | `for (auto& [k, v] : mappa)` |
| `{{ funzione arg1 arg2 }}` | chiamata (senza parentesi né virgole) | `funzione(arg1, arg2)` |
| `{{ arg \| funzione }}` | "pipe": passa `arg` come **ultimo** argomento | `funzione(..., arg)` |
| `{{ (funzione a) \| altra }}` | le parentesi raggruppano | |
| `{{/* commento */}}` | commento | `/* … */` |
| `{{- … -}}` | il trattino elimina gli spazi/a capo adiacenti | – |

"Vuoto" (falso) significa: `false`, `0`, `nil`, stringa vuota, lista o mappa vuota.

### Funzioni usate spesso in questo sito

| Funzione | Esempio | Note |
|---|---|---|
| `partial` | `{{ partial "icon.html" "cuore" }}` | il secondo argomento diventa il `.` del partial |
| `dict` | `(dict "url" . "title" $.Title)` | crea una mappa: per passare più parametri a un partial |
| `i18n` | `{{ i18n "readMore" }}` | stringa tradotta da `i18n/<lingua>.yaml` |
| `index` | `index $mappa "chiave"` | accesso a mappa/lista con chiave calcolata |
| `printf` | `printf "didascalia_%s" $lang` | come `sprintf` |
| `where` | `where site.RegularPages "Section" "eventi"` | filtra una collezione |
| `sort` | `sort $lista "Date" "asc"` | ordina |
| `first` | `first 3 $lista` | primi N elementi |
| `default` | `.Params.immagine \| default "/images/evento.svg"` | valore di ripiego |
| `time.Format` | `time.Format ":date_long" .Date` | data **localizzata** (`17 aprile 2027` / `17 April 2027`) |
| `relURL` / `relLangURL` | vedi §10 | percorsi con il prefisso del sito |
| `findRESubmatch` | vedi `youtube.html` | espressioni regolari (sintassi RE2) |
| `safeHTML` | `… \| safeHTML` | inserisce HTML senza escape (solo per contenuti fidati) |

### Escape automatico

Il motore fa l'escape automatico in base al contesto (HTML, attributo, URL, JavaScript), un po' come se ogni `<<` su uno stream applicasse la codifica giusta. Se un valore finisce in un URL "sospetto" viene sostituito con `#ZgotmplZ`: se lo vedi nell'HTML generato, è questo il motivo.

### Variabili globali utili

| | |
|---|---|
| `site.Language.Lang` | `it` / `en` (usato per scegliere dati e chiavi) |
| `site.Language.Locale` | `it-IT` / `en-GB` (usato in `lang`, `hreflang`, `og:locale`) |
| `site.Params.bozza` | parametro personalizzato in `hugo.toml` |
| `site.Menus.main` | voci del menu della lingua corrente |
| `site.RegularPages` | tutte le pagine "normali" della lingua corrente |
| `site.GetPage "/eventi"` | una pagina specifica, nella lingua corrente |
| `now` | data/ora **della build** |

---

## 9. Multilingua

### Configurazione (`hugo.toml`)

```toml
defaultContentLanguage = "it"
defaultContentLanguageInSubdir = false     # l'italiano è in /NES/, l'inglese in /NES/en/

[languages.it]
  label = "Italiano"
  locale = "it-IT"                         # formato date/numeri + attributi lang/hreflang
  contentDir = "content/it"
  weight = 1                               # ordine dei pulsanti IT/EN
[languages.en]
  label = "English"
  locale = "en-GB"
  contentDir = "content/en"
  weight = 2
```

Ogni lingua ha la propria cartella di contenuti, il proprio menu e i propri file `data/<lingua>/` e `i18n/<lingua>.yaml`.

### Come Hugo capisce che due pagine sono una la traduzione dell'altra

Due pagine di lingue diverse sono "traduzioni" se:
1. hanno **lo stesso percorso relativo** nelle due `contentDir` (es. `eventi/concerto-primavera.md` in entrambe), **oppure**
2. hanno lo **stesso `translationKey`** nel front matter (es. `chi-siamo.md` e `about.md`, entrambi `translationKey: about`).

Le traduzioni servono a:
- il pulsante **IT/EN** (`lang-switch.html`): porta alla traduzione della pagina corrente; se non esiste, alla home dell'altra lingua;
- i tag `<link rel="alternate" hreflang="…">` per Google (`head.html`).

### Stringhe fisse (`i18n/`)

Le scritte dei template ("Prossimi eventi", "Leggi tutto"…) non vanno scritte nell'HTML ma in `i18n/it.yaml` / `i18n/en.yaml` e richiamate con `{{ i18n "chiave" }}`.
Le chiavi `section_eventi`, `section_notizie`, `desc_eventi`, `desc_notizie` forniscono **titolo e descrizione** delle pagine di sezione (che non hanno un file `_index.md`, vedi `page-title.html` e `page-header.html`).

---

## 10. Indirizzi, percorsi e `baseURL`

Il sito è pubblicato in una **sottocartella**: `https://albertoscarduelli.github.io/NES/`. Questo prefisso `/NES/` è il punto che crea più errori quando si modifica a mano.

**Regola:** nei template non scrivere mai percorsi assoluti a mano (`href="/eventi/"`, `src="/images/x.svg"`). Usa sempre:

| Per | Usa | Risultato (IT) | Risultato (EN) |
|---|---|---|---|
| una risorsa (immagine, file) | `partial "url.html" "/images/x.svg"` | `/NES/images/x.svg` | `/NES/images/x.svg` |
| una pagina nella lingua corrente | `partial "link.html" "chi-siamo/"` | `/NES/chi-siamo/` | `/NES/en/chi-siamo/` |
| una pagina nota | `(site.GetPage "/eventi").RelPermalink` | `/NES/eventi/` | `/NES/en/events/` |
| la home della lingua | `site.Home.RelPermalink` | `/NES/` | `/NES/en/` |

`url.html` e `link.html` tolgono l'eventuale `/` iniziale e lasciano intatti i link esterni (`http…`, `mailto:`, `tel:`, `#…`).

> ⚠️ **Trappola:** non passare mai a `relURL` / `relLangURL` un percorso che **inizia con `/`**. Con una barra iniziale Hugo lo considera relativo alla radice del dominio e **scarta il prefisso `/NES/`**: per esempio `"/" | relLangURL` produce `/`, cioè `albertoscarduelli.github.io/` → errore 404. Per questo `url.html` e `link.html` tolgono la barra prima di chiamarle.
> In anteprima locale l'errore può passare inosservato, quindi dopo modifiche ai link conviene controllare l'HTML generato con il `baseURL` di produzione: `hugo --baseURL "https://albertoscarduelli.github.io/NES/"` e cerca in `public/` gli `href=/…` che non iniziano con `/NES/`.
Anche le immagini scritte nel Markdown (`![alt](/images/foto.jpg)`) passano da `url.html` grazie al *render hook* `layouts/_default/_markup/render-image.html`.

### Permalink in inglese

Le cartelle si chiamano `eventi` e `notizie` in **entrambe** le lingue (così le traduzioni si collegano da sole), ma in inglese l'indirizzo viene riscritto:

```toml
[languages.en.permalinks.page]
  eventi = "/events/:slug/"       # :slug = campo slug, oppure il titolo convertito
  notizie = "/news/:slug/"
[languages.en.permalinks.section]
  eventi = "/events/"
  notizie = "/news/"
```

### Menu

Il menu è definito in `hugo.toml`, per lingua, con `pageRef` (percorso del file **senza** `.md`, relativo a `contentDir`):

```toml
[[languages.it.menus.main]]
  name = "Chi siamo"
  pageRef = "/chi-siamo"
  weight = 20               # ordine
```

Con `pageRef` Hugo calcola l'URL corretto e sa evidenziare la voce attiva.

### Dominio personalizzato (in futuro)

Se l'associazione registra un dominio (es. `www.nes-associazione.it`):
1. cambia `baseURL` in `hugo.toml` → `"https://www.nes-associazione.it/"`;
2. su GitHub: *Settings → Pages → Custom domain* e configura il DNS come indicato da GitHub.
Grazie alla regola dei percorsi relativi, non serve toccare i template.

---

## 11. Eventi: prossimi e passati

- La data dell'evento è il campo `date`.
- `buildFuture = true` in `hugo.toml` è **indispensabile**: senza, Hugo scarterebbe le pagine con data futura.
- La divisione avviene **a build-time** confrontando `.Date` con `now.AddDate 0 0 -1` (ieri, così un evento resta tra i "prossimi" per tutto il suo giorno):

```html
{{ $cut := now.AddDate 0 0 -1 }}
{{ $up   := sort (where .RegularPages "Date" "ge" $cut) "Date" "asc" }}
{{ $past := sort (where .RegularPages "Date" "lt" $cut) "Date" "desc" }}
```

- Siccome `now` è l'ora della build, la pipeline viene rieseguita **ogni notte** (`schedule` nel workflow, §15): così un evento passato si sposta da solo anche se nessuno modifica il sito.
- Fuso orario: la build gira con `TZ: Europe/Rome`.

---

## 12. Video YouTube

`layouts/partials/youtube.html`, richiamato da `eventi/single.html`, `notizie/single.html` e `gallery.html`:

```html
{{ partial "youtube.html" (dict "url" .Params.video "title" .Title) }}
```

1. Estrae l'ID del video (11 caratteri) dal link con una regex RE2: accetta `watch?v=`, `youtu.be/`, `/shorts/`, `/embed/`, `/live/`, oppure l'ID da solo. Se l'ID non si trova, non mostra nulla.
2. Genera un **segnaposto** (sfondo verde, pulsante ▶, titolo). Nessuna risorsa viene chiesta a YouTube/Google: nemmeno la miniatura, che sarebbe già un trasferimento di dati.
3. Al clic, `main.js` sostituisce il segnaposto con un `<iframe>` di `https://www.youtube-nocookie.com/embed/<ID>?autoplay=1&rel=0`.

È un approccio "two-click" pensato per il GDPR. Le schede di eventi e notizie mostrano l'etichetta **▶ Video** se il campo `video` è presente.

---

## 13. CSS e JavaScript

### CSS (`assets/css/style.css`)

- Un unico file, diviso in sezioni commentate (header, pulsanti, hero, schede, galleria, contatti, footer, video, media query).
- **Variabili** in `:root` all'inizio del file: colori, raggi, ombre, font, larghezza massima. Cambiando quelle si ricolora tutto il sito.

| Variabile | Uso |
|---|---|
| `--c-primary`, `--c-primary-dark`, `--c-primary-soft` | verde petrolio: menu attivo, fasce, icone |
| `--c-accent` | ambra: pulsanti principali, dettagli |
| `--c-ink`, `--c-muted` | testo principale e secondario |
| `--c-bg`, `--c-surface`, `--c-alt` | sfondi: pagina, schede, sezioni alternate |
| `--c-footer` | sfondo del piè di pagina |
| `--radius`, `--shadow` | arrotondamenti e ombre |
| `--max` | larghezza massima del contenuto (1180px) |

- **Responsive**: le regole `@media (max-width: …)` / `(min-width: …)` cambiano l'impaginazione in base alla larghezza dello schermo. Soglie usate: 600px (telefono), 720px (griglia a 3 colonne), 860px (contatti/footer), 1000px (menu a scomparsa).
- Nessun font esterno (si usa il font di sistema): niente richieste a Google Fonts, quindi nessun problema GDPR e pagine più veloci.

Il template lo include così:

```html
{{ $css := resources.Get "css/style.css" | minify | fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```

`fingerprint` aggiunge un hash al nome (`style.min.<hash>.css`): a ogni modifica cambia il nome, quindi i browser non usano una versione vecchia in cache.

### JavaScript (`assets/js/main.js`)

JavaScript "vanilla", senza librerie, solo miglioramenti progressivi (il sito funziona anche senza JS, tranne questi comportamenti):

| Funzione | Elementi coinvolti |
|---|---|
| apertura/chiusura del menu su smartphone (anche con Esc) | `.nav-toggle`, `#nav` (classe `open`) |
| ombra sotto l'intestazione quando si scorre | `.site-header` (classe `scrolled`) |
| lightbox della galleria (`<dialog>` nativo) | `.lightbox-link`, `.lightbox` |
| avvio dei video YouTube al clic | `.video-frame`, `.video-play` |

---

## 14. Pannello di modifica Pages CMS

[Pages CMS](https://pagescms.org) è un servizio web gratuito che legge `.pages.yml` e mostra dei **moduli** per modificare i file del repository; al salvataggio fa un commit su GitHub, che avvia la pubblicazione.

Struttura di `.pages.yml`:

```yaml
media:
  input: static/images        # dove salva le immagini caricate
  output: /images             # come le scrive nei campi (→ partial url.html aggiunge /NES)

content:
  - name: eventi-it           # identificatore
    label: "Eventi – Italiano"  # nome mostrato
    type: collection          # collection = cartella con tanti file; file = un singolo file
    path: content/it/eventi
    filename: "{year}-{month}-{day}-{primary}.md"   # nome dei nuovi file
    fields: &eventFields      # &nome = "ancora" YAML, riutilizzata con *eventFields
      - { name: title, label: "Titolo", type: string, required: true }
      - { name: date,  label: "Data",   type: date }
      - { name: body,  label: "Testo completo", type: rich-text }   # body = corpo del Markdown
```

- Ogni **campo** del modulo corrisponde a un campo del front matter (o di un file YAML); `body` è il testo sotto il front matter.
- Le ancore YAML (`&homeFields` / `*homeFields`) evitano di ripetere gli stessi campi per IT e EN.
- **Se aggiungi un campo a un tipo di contenuto, aggiungilo in tre posti:** nel front matter dei file esistenti (facoltativo), nel template che lo mostra, e in `.pages.yml` (per renderlo modificabile dal pannello).
- Il campo `translationKey` delle pagine è `hidden: true`: serve a non farlo modificare per errore (Galleria e Contatti dipendono da quel valore, §7).

> Il comportamento esatto di alcune opzioni (per esempio `hidden`, `subfolders`, `view`) dipende dalla versione di Pages CMS: se qualcosa non appare come previsto, consulta la [documentazione ufficiale](https://pagescms.org/docs/).

---

## 15. Pubblicazione: GitHub Actions e GitHub Pages

File: `.github/workflows/hugo.yml`.

| Quando parte | Perché |
|---|---|
| `push` sul ramo `main` | ogni modifica (anche da Pages CMS o dal sito di GitHub) |
| `schedule: "0 3 * * *"` | ogni notte alle 03:00 **UTC**, per aggiornare eventi prossimi/passati |
| `workflow_dispatch` | avvio manuale dalla scheda *Actions* (pulsante *Run workflow*) |

Passi del job `build`:
1. installa Hugo **extended** alla versione fissata in `HUGO_VERSION` (oggi `0.169.0`, la stessa usata in locale);
2. scarica il repository;
3. `configure-pages` restituisce l'URL pubblico del sito;
4. `hugo --gc --minify --baseURL "<url>/"` genera `public/`;
5. carica `public/` come artefatto; il job `deploy` lo pubblica su GitHub Pages.

Note:
- **Configurazione iniziale (una volta):** *Settings → Pages → Build and deployment → Source: GitHub Actions*.
- Stato e log di ogni pubblicazione: scheda **Actions** del repository (✔ verde = pubblicato, ✖ rosso = errore, il sito resta alla versione precedente).
- GitHub disattiva i workflow pianificati (`schedule`) dopo **60 giorni senza attività** nel repository: se succede, compare un avviso nella scheda Actions con il pulsante per riattivarli.
- **Aggiornare Hugo:** cambiare `HUGO_VERSION` nel workflow e aggiornare anche la copia locale (`winget upgrade Hugo.Hugo.Extended`). Prima di farlo prova la build in locale: le versioni nuove possono deprecare funzioni (vedi i messaggi `WARN deprecated`).

---

## 16. Lavorare in locale

### Requisiti

- **Hugo extended** (installato con `winget install Hugo.Hugo.Extended`).
- **Git**.
- Un editor; consigliato VS Code con le estensioni "Hugo Language and Syntax Support" e "YAML".

### Comandi

```bash
hugo server                 # anteprima su http://localhost:1313/NES/ con ricarica automatica
hugo server --navigateToChanged   # il browser va sulla pagina appena modificata
hugo                        # build completa in public/ (come in produzione)
hugo --printPathWarnings    # avvisa se due file producono lo stesso indirizzo
```

Se il comando `hugo` non viene trovato subito dopo l'installazione, riapri il terminale (winget aggiorna il `PATH`).

### Flusso di lavoro con git

```bash
git pull                                   # 1. prendi le modifiche fatte da altri (Pages CMS, GitHub)
hugo server                                # 2. modifica e controlla l'anteprima
git add -A                                 # 3. prepara le modifiche
git commit -m "Descrizione della modifica" # 4. registra
git push                                   # 5. invia: parte la pubblicazione
```

**Fai sempre `git pull` prima di iniziare**: chi usa Pages CMS fa commit direttamente su GitHub, e senza pull i due rami divergono.

L'identità git di questo repository è configurata **solo localmente** (`git config user.name` / `user.email` senza `--global`): non interferisce con altri progetti.

Gli avvisi `LF will be replaced by CRLF` di git su Windows sono innocui.

---

## 17. Ricette: modifiche comuni

### Cambiare i colori
`assets/css/style.css`, variabili in `:root` (§13). Per coerenza cambia anche `theme-color` in `layouts/partials/head.html` (colore della barra del browser sui telefoni).

### Cambiare il logo
Sostituisci `static/images/logo.svg` (o carica un'altra immagine e cambia `logo:` in `data/associazione.yaml`). Formato consigliato: SVG o PNG quadrato, almeno 256×256. Il logo è usato anche come icona della scheda del browser.

### Aggiungere una pagina semplice (es. "Statuto")
1. `content/it/statuto.md` con `title`, `description`, `translationKey: statuto` e il testo.
2. `content/en/statute.md` con lo stesso `translationKey: statuto`.
3. Voce di menu in `hugo.toml` per entrambe le lingue:
   ```toml
   [[languages.it.menus.main]]
     name = "Statuto"
     pageRef = "/statuto"
     weight = 65
   [[languages.en.menus.main]]
     name = "Statute"
     pageRef = "/statute"
     weight = 65
   ```
4. Niente da fare in `.pages.yml`: la pagina compare da sola nelle collezioni "Pagine".

### Togliere o riordinare voci di menu
`hugo.toml`: cancella il blocco `[[languages.xx.menus.main]]` o cambia `weight` (più basso = più a sinistra). Ricorda entrambe le lingue.

### Aggiungere un campo agli eventi (es. "costo")
1. Template: in `layouts/eventi/single.html`, dentro `<ul class="event-facts">`:
   ```html
   {{ with .Params.costo }}<li>{{ partial "icon.html" "stella" }}<span>{{ . }}</span></li>{{ end }}
   ```
2. Pannello: in `.pages.yml`, nell'elenco `&eventFields`:
   ```yaml
   - { name: costo, label: "Costo (es. Ingresso libero / 5 €)", type: string }
   ```
3. Contenuti: aggiungi `costo: "5 €"` agli eventi che lo prevedono.

### Aggiungere una nuova sezione (es. "Progetti", con un file per progetto)
1. Cartelle `content/it/progetti/` e `content/en/progetti/` con i file.
2. Template: copia `layouts/notizie/` in `layouts/progetti/` e adatta.
3. Stringhe: `section_progetti` e `desc_progetti` in entrambi i file `i18n/`.
4. Permalink inglese in `hugo.toml` (`progetti = "/projects/:slug/"` nelle due tabelle `permalinks`).
5. Menu in `hugo.toml` (`pageRef = "/progetti"` in entrambe le lingue).
6. Due collezioni in `.pages.yml` (copia quelle delle notizie).

### Aggiungere un'icona
In `layouts/partials/icon.html` aggiungi una coppia `"nome" `<path …/>`` al `dict`. Le icone sono SVG 24×24 a tratto (`stroke`): si possono prendere da librerie con licenza libera come [Feather](https://feathericons.com) o [Lucide](https://lucide.dev), copiando solo gli elementi interni a `<svg>`. Per renderla selezionabile nei riquadri della home, aggiungi il nome alla lista `values` del campo `icona` in `.pages.yml`.

### Modificare una scritta fissa
`i18n/it.yaml` e `i18n/en.yaml` (es. `readMore: Leggi tutto`).

### Cambiare il numero di eventi/notizie in home o per pagina
- Home: `first 3` in `layouts/index.html`.
- Notizie per pagina: `pagerSize` in `hugo.toml`.

### Aggiungere una terza lingua (es. francese)
1. `hugo.toml`: blocco `[languages.fr]` (`label`, `locale = "fr-FR"`, `contentDir = "content/fr"`, `weight = 3`, menu e permalink).
2. `content/fr/` con le pagine (stessi percorsi o `translationKey`).
3. `data/fr/home.yaml`, `i18n/fr.yaml`.
4. Campi `_fr` in `data/associazione.yaml` e `data/galleria.yaml` (`descrizione_fr`, `didascalia_fr`, `titolo_fr`).
5. Collezioni `fr` in `.pages.yml`.
Il pulsante lingua si aggiorna da solo.

### Togliere la modalità "bozza" (rendere il sito visibile a Google)
`hugo.toml` → `bozza = false`. Rimuove il meta tag `noindex, nofollow`.

---

## 18. Debug e problemi frequenti

### Strumenti

| Cosa | Come |
|---|---|
| Errori di build | `hugo server` li mostra nel terminale **e** nel browser (pagina rossa con file e riga) |
| Vedere il valore di una variabile | `{{ warnf "DEBUG %#v" $x }}` → stampa nel terminale; oppure `<pre>{{ debug.Dump $x }}</pre>` nell'HTML |
| Vedere l'HTML generato | `hugo` e apri i file in `public/`, oppure "Visualizza sorgente" nel browser |
| Problemi di stile/JS | strumenti per sviluppatori del browser (F12): scheda *Elements* per il CSS, *Console* per gli errori JS |
| Errori in produzione | scheda **Actions** su GitHub → l'esecuzione fallita → passo "Genera il sito" |

### Problemi tipici

| Sintomo | Causa probabile |
|---|---|
| La build fallisce dopo una modifica a un `.yaml` o al front matter | indentazione sbagliata, tab al posto degli spazi, `:` non tra virgolette |
| Un evento futuro non compare | data scritta male, oppure `buildFuture` tolto da `hugo.toml` |
| Un evento passato è ancora tra i "prossimi" | il sito non è stato ricostruito: controlla che il workflow notturno sia attivo (Actions) |
| Un'immagine non si vede online ma in locale sì | percorso scritto a mano senza `url.html`, oppure maiuscole/minuscole diverse (GitHub distingue `Foto.JPG` da `foto.jpg`, Windows no) |
| Il pulsante IT/EN porta alla home invece che alla pagina tradotta | manca la traduzione, o il `translationKey` è diverso nelle due lingue |
| Nell'HTML compare `#ZgotmplZ` | un valore non ritenuto sicuro in un URL/attributo (§8, escape automatico) |
| `WARN deprecated: …` | funzione o chiave deprecata in una versione nuova di Hugo: aggiorna come suggerito nel messaggio |
| Modifiche al CSS non visibili | cache del browser: Ctrl+F5 (in produzione il fingerprint lo evita) |
| `git push` rifiutato ("fetch first") | qualcuno ha modificato il sito da Pages CMS/GitHub: `git pull` e poi di nuovo `git push` |

### Tornare indietro

Ogni modifica è un commit: si può sempre tornare a una versione precedente.
- **Da GitHub:** apri il file → *History* → scegli la versione → copia il contenuto.
- **Da git:** `git log` per trovare il commit, `git revert <commit>` per annullarlo creando un nuovo commit (sicuro, non riscrive la storia).

---

## 19. Riferimenti

| Argomento | Link |
|---|---|
| Hugo – documentazione | https://gohugo.io/documentation/ |
| Hugo – funzioni dei template | https://gohugo.io/functions/ |
| Hugo – multilingua | https://gohugo.io/content-management/multilingual/ |
| Hugo – ordine di ricerca dei template | https://gohugo.io/templates/lookup-order/ |
| Go template (sintassi di base) | https://pkg.go.dev/text/template |
| Pages CMS – documentazione | https://pagescms.org/docs/ |
| GitHub Pages | https://docs.github.com/pages |
| GitHub Actions | https://docs.github.com/actions |
| Markdown (guida rapida) | https://www.markdownguide.org/cheat-sheet/ |
| CSS / HTML / JS (riferimento) | https://developer.mozilla.org/it/ |
