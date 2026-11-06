# Sito web dell'associazione NES

Indirizzo del sito: **https://albertoscarduelli.github.io/NES/** (versione inglese: `/NES/en/`)

Sito statico bilingue (italiano / inglese) generato con [Hugo](https://gohugo.io), pubblicato automaticamente su GitHub Pages e modificabile dal browser con il pannello [Pages CMS](https://app.pagescms.org).

## Documentazione

| Documento | Per chi |
|---|---|
| 📝 [**Guida all'aggiornamento**](docs/GUIDA-AGGIORNAMENTO.md) | chi aggiorna eventi, notizie, foto, video e testi (nessuna competenza tecnica richiesta) |
| 🛠️ [**Documentazione tecnica**](docs/ARCHITETTURA.md) | chi modifica a mano grafica, struttura e configurazione |

## In breve

Il sito ha due parti ben separate:

| Parte | Dove si trova | Chi la tocca |
|---|---|---|
| **Contenuti**: testi, eventi, notizie, foto, video, contatti | `content/`, `data/`, `static/images/` | chiunque aggiorni il sito |
| **Struttura e grafica**: impaginazione, colori, menu | `layouts/`, `assets/`, `i18n/`, `hugo.toml` | solo chi sviluppa il sito |

Dopo ogni modifica salvata su GitHub il sito si aggiorna da solo in 1–2 minuti, e si ricostruisce anche ogni notte (per spostare gli eventi passati).

## Avvio rapido per lo sviluppatore

```bash
git pull
hugo server        # anteprima su http://localhost:1313/NES/
```

Configurazione iniziale di GitHub Pages (una volta sola): *Settings → Pages → Build and deployment → Source: **GitHub Actions***.

Il sito è in modalità **bozza** (`bozza = true` in `hugo.toml`): i motori di ricerca non lo indicizzano finché non viene messo a `false`.
