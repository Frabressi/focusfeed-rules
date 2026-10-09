# Formato regole FocusFeed (v1)

File: `rules/v1/<app>.json` (attualmente `instagram.json`).

```json
{
  "version": 1,
  "minAppVersion": 1,
  "updatedAt": "2026-10-08T00:00:00Z",
  "rules": [],
  "thirdPartyBlocklist": ["doubleclick.net"]
}
```

- `version`: intero >= 1. L'app accetta un aggiornamento solo se `version` >= quella corrente.
- `minAppVersion`: versionCode minimo dell'app; se superiore, il file viene ignorato.
- `updatedAt`: data ISO-8601.
- `thirdPartyBlocklist`: domini; bloccati (anche sottodomini) solo le richieste verso questi host.
  Mai bloccati: `*.instagram.com`, `*.cdninstagram.com`, `*.fbcdn.net`, `*.facebook.com`, `*.facebook.net`.

## Rule

| campo | tipo | note |
|---|---|---|
| `id` | string | obbligatorio, univoco |
| `enabled` | bool | default `true` |
| `type` | `hide_css` \| `hide_js` \| `redirect` \| `style` \| `hide_has_text` \| `block_overlay` \| `rewrite_nav` | obbligatorio |
| `urlPattern` | regex | opzionale; match (find) sull'URL della pagina; default tutte |
| `selector` | string | `hide_css`: selettore CSS, applicato come `display:none!important`; `hide_has_text`: elementi candidati |
| `css` | string | `style`: CSS arbitrario |
| `js` | string | `hide_js`: corpo di funzione con parametro `root` che restituisce `Element[]` da nascondere |
| `from` | regex | `redirect`: pattern sull'URL di navigazione |
| `to` | string | `redirect`: sostituzione (`$1`...); il risultato deve iniziare con `https://`. `rewrite_nav`: destinazione, inizia con `/` o `https://www.instagram.com/` |
| `text` | string[] | `hide_has_text`: testi cercati (case-insensitive), almeno uno, non vuoti |
| `match` | `exact` \| `contains` | `hide_has_text`: confronto sul `textContent` (trim) dei discendenti foglia; default `exact` |
| `within` | string | `hide_has_text`: selettore opzionale dei discendenti da esaminare (default tutti) |
| `mode` | `hide` \| `cover` | `hide_has_text`: `hide` (default) = `display:none` sul candidato; `cover` = vedi sotto |
| `message` | string | `block_overlay`: testo mostrato (default "Contenuto bloccato") |
| `comment` | string | libero |

### Tipi dichiarativi (senza eval)

`hide_js` usa `new Function` e **non funziona** su instagram.com (CSP senza `'unsafe-eval'`): preferire i tipi seguenti.

- `hide_has_text`: nasconde (`display:none!important` inline) ogni elemento `selector` che contiene un discendente
  foglia (nessun elemento figlio), eventualmente ristretto a `within`, il cui testo corrisponde a uno di `text`.
  Rivalutato a ogni mutazione del DOM e a ogni cambio URL (anche SPA).
  Con `mode: "cover"` il candidato non viene nascosto: riceve l'attributo `data-ff-cover` (rimosso se l'etichetta
  sparisce, es. storia successiva) e il motore aggiunge (una volta) solo CSS: un `::after` a schermo intero
  (`position:fixed`, sfondo nero, testo "Inserzione nascosta — tocca a destra per andare avanti",
  `pointer-events:none`) e `opacity:0` su tutti i discendenti (non `visibility:hidden`, per non togliere ai tap
  il bersaglio che Instagram usa per avanzare). I tap dell'utente passano quindi a Instagram, che avanza/chiude la
  storia normalmente. Nessun click simulato, timer, fetch o scroll. Es. storie sponsorizzate:
  `{"type":"hide_has_text","urlPattern":"^https://www\.instagram\.com/stories/","selector":"section","within":"header *","text":["Inserzione","Sponsored"],"mode":"cover"}`.
- `block_overlay`: richiede `urlPattern`. Se l'URL corrente (anche dopo `pushState`/`replaceState`/`popstate`)
  matcha, mostra un overlay fisso a schermo intero (solo DOM aggiunto dall'app) con `message` e il pulsante
  "Torna al feed" (`history.back()`, altrimenti `/?variant=following`, solo al tap dell'utente); il `body`
  della pagina viene nascosto finche' l'overlay e' attivo. Vince la prima regola che matcha.

- `rewrite_nav`: richiede `selector` e `to`. Listener `click` in fase di capture: se il click e' un tap reale
  dell'utente (`event.isTrusted`, mai simulato) su un elemento che matcha `selector` (e `urlPattern`, se presente),
  fa `preventDefault` e `location.assign(to)`. Es. Home di Instagram `a[href="/"]` -> `/?variant=following`.

Esempi:

```json
{"id":"sp","type":"hide_has_text","selector":"article","text":["Sponsored","Sponsorizzato"],"match":"exact"}
{"id":"nv","type":"rewrite_nav","selector":"a[href=\"/\"]","to":"/?variant=following"}
{"id":"rl","type":"block_overlay","urlPattern":"^https://www\.instagram\.com/reel/","message":"Reel bloccato."}
```

Campi sconosciuti vengono ignorati. Le regole invalide (regex non compilabile, campi richiesti mancanti,
tipo sconosciuto) vengono scartate singolarmente senza invalidare il file.

Le regole nascondono solo visivamente (`display:none`): nessun click, scroll, fetch o modifica di richieste.
`redirect` si applica solo alle navigazioni top-level dentro l'app.
