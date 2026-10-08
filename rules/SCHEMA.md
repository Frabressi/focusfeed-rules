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
| `type` | `hide_css` \| `hide_js` \| `redirect` \| `style` | obbligatorio |
| `urlPattern` | regex | opzionale; match (find) sull'URL della pagina; default tutte |
| `selector` | string | `hide_css`: selettore CSS, applicato come `display:none!important` |
| `css` | string | `style`: CSS arbitrario |
| `js` | string | `hide_js`: corpo di funzione con parametro `root` che restituisce `Element[]` da nascondere |
| `from` | regex | `redirect`: pattern sull'URL di navigazione |
| `to` | string | `redirect`: sostituzione (`$1`...); il risultato deve iniziare con `https://` |
| `comment` | string | libero |

Campi sconosciuti vengono ignorati. Le regole invalide (regex non compilabile, campi richiesti mancanti,
tipo sconosciuto) vengono scartate singolarmente senza invalidare il file.

Le regole nascondono solo visivamente (`display:none`): nessun click, scroll, fetch o modifica di richieste.
`redirect` si applica solo alle navigazioni top-level dentro l'app.
