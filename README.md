# Frappe ERP Stack

Előre buildelt, élesre szabott Docker image, amely az **ERPNext v16-ot és a Frappe ökoszisztéma kilenc további hivatalos appját** tartalmazza egyetlen, telepítésre kész csomagban. A buildet a GitHub Actions végzi automatikusan, így a kész image a publikus GitHub Container Registryből (`ghcr.io`) anonim módon, kulcsfordítás nélkül letölthető.

[![Build and Push Image](https://github.com/szricsi0917/frappe-erp-stack/actions/workflows/build.yml/badge.svg)](https://github.com/szricsi0917/frappe-erp-stack/actions/workflows/build.yml)

## Miért érdemes ezt használni?

A Frappe csapat hivatalos `frappe/erpnext` Docker imageje csak a `frappe` keretrendszert és magát az ERPNext-et tartalmazza. Bármely további Frappe app (HRMS, Helpdesk, CRM stb.) használatához saját Docker imaget kell buildelni, egy `apps.json` megadásával. A build maga nem bonyolult, de:

- Minden upstream patch megjelenésekor újra kell futtatni.
- A teljes build 6–8 GB méretű, és egy átlagos szerveren 25–40 percig tart.
- Sok memóriát és átmeneti tárhelyet igényel, ami egy production szerveren felesleges terhelés.

Ez a repó a fenti folyamatot automatizálja. A build a GitHub felhőszerverein fut, az eredmény image-t te csak letöltöd:

```bash
docker pull ghcr.io/szricsi0917/frappe-erp-stack:v16
```

## Tartalom

Az image-en jelenleg az alábbi 11 hivatalos Frappe app van telepítve:

| App | Branch | Funkció |
|-----|--------|---------|
| frappe | version-16 | A Frappe alap keretrendszer |
| erpnext | version-16 | Értékesítés, beszerzés, raktár, gyártás, könyvelés, projektek, eszközök |
| hrms | version-16 | Részletes HR – bér, szabadság, jelenlét, költségelszámolás, műszakkezelés |
| helpdesk | develop | Ticketrendszer SLA-val, ügyfélportállal |
| insights | develop | BI / dashboard / riport eszköz a Frappe adatok felett |
| builder | develop | Vizuális, kódolás nélküli weboldal-szerkesztő |
| wiki | develop | Belső tudásbázis |
| lms | develop | Frappe Learning – online oktatási rendszer |
| gameplan | develop | Csapatkommunikáció és projektdiszkusszió |
| print_designer | develop | Drag-and-drop nyomtatási sablonszerkesztő |
| crm | develop | Önálló CRM modern felülettel |

Az aktuális applista mindig az [`apps.json`](apps.json) fájlban található.

## Image cimkék

| Tag | Jelentés |
|-----|----------|
| `:v16` | Mindig a legfrissebb sikeres v16-os build. **Ezt használd alapértelmezetten.** |
| `:latest` | Alias a `:v16`-ra. |
| `:<commit-sha>` | Egy konkrét git commitra rögzített, immutábilis build. Visszaállításhoz hasznos. |

## Gyors indulás

### Előfeltételek

- Docker Engine 24+
- Docker Compose v2
- Legalább 4 GB szabad RAM és 20 GB tárhely

### Lépések

1. **Klónozd a repót** (vagy csak töltsd le a `docker-compose.yml` és `.env.example` fájlokat):

   ```bash
   git clone https://github.com/szricsi0917/frappe-erp-stack.git
   cd frappe-erp-stack
   ```

2. **Hozd létre a `.env` fájlt** a sablonból, majd töltsd ki:

   ```bash
   cp .env.example .env
   nano .env   # vagy bármely más szerkesztő
   ```

   Minimum ezeket állítsd be: `SITE_NAME`, `DB_PASSWORD`, `ADMIN_PASSWORD`.

3. **Indítsd el a rendszert:**

   ```bash
   docker compose up -d
   ```

4. **Várj 5–10 percet**, amíg a `create-site` konténer befejezi a site létrehozását és az appok telepítését:

   ```bash
   docker compose logs -f create-site
   ```

   A „Kész." üzenetnél van vége.

5. **Lépj be a rendszerbe** a böngészőben: `http://<host>:8080`
   - Felhasználónév: `Administrator`
   - Jelszó: amit az `ADMIN_PASSWORD`-ba írtál

## Saját app-lista (forkolás)

Ha más app-kombinációt szeretnél (más appokat, más branch-eket), forkold a repót, és módosítsd az `apps.json` fájlt. A formátum:

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-16"
  },
  {
    "url": "https://github.com/sajat-szervezet/custom-app",
    "branch": "main"
  }
]
```

A `main` branchre pusholva a GitHub Actions automatikusan elindít egy buildet. A kész image a saját forkod GHCR Packages oldalán jelenik meg.

A `docker-compose.yml`-ben módosítsd az `IMAGE` változót a `.env`-ben az új imagedre:

```env
IMAGE=ghcr.io/sajat-githubon/frappe-erp-stack
```

## Build folyamat

A workflow ([`.github/workflows/build.yml`](.github/workflows/build.yml)) három esetben indul:

1. **Automatikusan**, amikor az `apps.json` vagy a workflow-fájl módosul a `main` branchen.
2. **Heti rendszerességgel** (vasárnap 03:00 UTC), hogy a friss upstream patchek mindig bekerüljenek.
3. **Manuálisan** az Actions fülről, a „Run workflow" gombbal.

Egy teljes build kb. 25–40 percig tart.

## Production deployment

A `docker-compose.yml` template csak demonstrációra van beállítva (helyi mappákba mountol). Production rendszerre az alábbi módosításokat ajánljuk a `.env`-ben:

- **Külön tárhely a tartós adatoknak:** állítsd a `SITES_DIR`, `DB_DIR`, `UPLOADS_*` változókat dedikált disk mountokra (NE az alapértelmezett `./data` mappára).
- **Erős jelszavak:** `DB_PASSWORD` és `ADMIN_PASSWORD` minimum 16 karakter, vegyes karakterekkel.
- **Reverse proxy + HTTPS:** a `HTTP_PORT`-ot egy belső portra állítsd, és tegyél elé egy reverse proxyt (Caddy, Traefik, nginx) Let's Encrypt tanúsítvánnyal. A `/socket.io/` útvonalat is proxyzni kell a websocket frissítésekért.
- **Backupok:** a `frappe-bench/sites/<sitename>/private/backups` mappa kerüljön rendszeres mentésre.
- **Memória:** ha a gép 8 GB RAM alatt van, csökkentsd az `INNODB_BUFFER_POOL_SIZE`-t 1G-re.

A Frappe Docker dokumentáció további részleteket ad reverse proxyról, monitoringról, mentésekről:
https://github.com/frappe/frappe_docker

## Privát használat

Ha privát image-ként szeretnéd futtatni (pl. mert saját, nem nyilvános appot is buildelsz):

1. Forkold a repót **privátként**.
2. A GHCR-en állítsd a package láthatóságát privátra (`Package settings → Change visibility → Private`).
3. A futtató rendszereden jelentkezz be a `ghcr.io`-ra egy [GitHub Personal Access Token](https://github.com/settings/tokens)-nel (`read:packages` scope):

   ```bash
   echo $GITHUB_TOKEN | docker login ghcr.io -u <githubon> --password-stdin
   ```

A GitHub jelenleg ingyenesen biztosítja a privát Container Registry tárolást és letöltést is, ez azonban a szolgáltató döntése alapján bármikor változhat. Hosszú távon a publikus image biztonságosabb választás.

## Hozzájárulás

Issue-kat és pull request-eket szívesen fogadunk. Mielőtt megnyitnál egyet:

- **Új app hozzáadására** vonatkozó kérés: nyiss issue-t, ahol leírod, miért hasznos lehet általánosan.
- **Branch módosításra** vonatkozó kérés: indokold meg a kompatibilitási megfontolásokat.

Note: ez a repó saját, személyes / céges használatra készült, és a Frappe csapat hivatalos image-ének kiegészítése – nem versenytársa annak.

## License

MIT – lásd a [LICENSE](LICENSE) fájlt.

A felhasznált Frappe és ERPNext szoftverek a saját licenszeik alatt állnak (GPLv3 és AGPLv3). Ez a repó csak a build folyamatot és a konfigurációt tartalmazza; nem módosítja a fenti szoftvereket.
