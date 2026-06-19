# hegyesihq-engine

Sajat ERPNext v16 image a HegyesiHQ telekom-rendszerhez.
Build: GitHub Actions -> push -> GHCR (`ghcr.io/szricsi0917/hegyesihq-engine`).
Deploy: Unraid + Portainer (standalone Docker).

## Telepitett app-ok

| App | Branch | Cel |
|---|---|---|
| erpnext | version-16 | ERP magmodulok |
| hrms | version-16 | HR / fizetes / szabadsag |
| helpdesk | main | Ticketrendszer |
| insights | main | BI / dashboard |
| builder | main | No-code page builder |
| wiki | master | Belso tudasbazis |
| lms | main | Frappe Learning – oktatas |
| gameplan | main | Csapatkommunikacio |
| print_designer | main | Magyar szamla/munkalap sablonok |
| crm | main | Onallo CRM |

App-ok modositasa: `apps.json` szerkesztese a fo branchen, commit -> GH Actions automatikusan rebuildel.

## Image cimkek

- `:v16` – ezt hasznald a compose-ban
- `:latest` – ugyanaz, alias
- `:<sha>` – konkret commitra rogzitett (visszaallashoz)

## Build trigger

- `apps.json` vagy a workflow modositasa -> auto build
- Heti automatikus rebuild (vasarnap 03:00 UTC) – patchek begyujtese
- Kezi inditas: GitHub repo -> Actions ful -> "Build and Push Image" -> "Run workflow"

## Unraid-en pulloldas privat GHCR-bol

1. GitHub -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic)
   -> Generate new token: scope `read:packages`, no expiration (vagy hosszu)
   -> token ertek mentese!
2. Portainer -> Registries -> Add registry:
   - Provider: Custom registry
   - URL: `ghcr.io`
   - Authentication: ON
   - Username: `szricsi0917`
   - Password: a tokent ide
3. A stack telepiteseig nincs mas teendo – Portainer automatikusan a fenti credential-lel huzza.

## Deploy

Lasd `docker-compose.yml`. Portainer Web Editorba bemasolod, az Environment variables
szekcioba kerul: `SITE_NAME`, `DB_PASSWORD`, `ERPNEXT_ADMIN_PASSWORD`.

Image tag a compose-ban kozvetlenul: `ghcr.io/szricsi0917/hegyesihq-engine:v16`.

A `create-site` szolgaltatas idempotens: meglevo site-ot nem irja felul, csak a
hianyzo app-okat installalja es lefuttat egy migrate-et minden deploykor.
