# memura-landing — www.memuraapp.it

Landing pubblica di Memura, ospitata su **GitHub Pages** del repo
`tech-memuraapp/memura-landing` ed esposta su `https://www.memuraapp.it/`
via custom domain. Stesso pattern di `memura-legal` (vedi
`memura/src/config/legal.ts`).

## Struttura

```
memura-landing/
├── index.html        # landing IT+EN (toggle client-side)
├── CNAME             # www.memuraapp.it → GH Pages legge per custom domain
├── robots.txt
├── sitemap.xml
└── assets/
    ├── favicon.svg
    └── og-image.svg  # da convertire in PNG per OG (vedi sotto)
```

> Niente `.htaccess` — GitHub Pages è nginx-like, non lo legge. HTTPS,
> redirect www↔apex e cache headers sono gestiti automaticamente da GH Pages
> (Fastly CDN davanti).

---

## 1. Crea repo su GitHub

Da `tech-memuraapp`:

1. Account → **New repository** → nome `memura-landing` → **Public**
   (Pages gratis solo su public per account non-Pro).
2. **Non** spuntare README/`.gitignore` — già presenti localmente.
3. Crea il repo vuoto.

## 2. Push iniziale

Dalla cartella `memuraapp-landing/` sul disco:

```bash
cd ~/Desktop/Memura/memuraapp-landing

git init -b main
git add .
git commit -m "feat: landing initial — www.memuraapp.it"
git remote add origin git@github.com:tech-memuraapp/memura-landing.git
git push -u origin main
```

Se usi HTTPS invece di SSH: `git remote add origin https://github.com/tech-memuraapp/memura-landing.git`.

## 3. Attiva GitHub Pages

Repo → **Settings** → **Pages**:

- **Source**: `Deploy from a branch`
- **Branch**: `main` → `/ (root)` → **Save**
- Attendi 30-60s, GH Pages builda.
- Sezione **Custom domain**: il file `CNAME` nella root del repo viene
  letto automaticamente → mostrerà `www.memuraapp.it`.
- Sezione **Enforce HTTPS**: spunta dopo che il cert Let's Encrypt è
  provisionato (può richiedere fino a 24h dopo che DNS è propagato).

## 4. DNS su register.it

Pannello register.it → dominio `memuraapp.it` → **DNS / Zona DNS**.

### 4a. CNAME per `www` → GitHub Pages

```
Tipo:   CNAME
Nome:   www
Valore: tech-memuraapp.github.io.
TTL:    3600
```

> Il punto finale dopo `.io` è opzionale ma corretto: indica FQDN assoluto.

### 4b. A records sull'apex `memuraapp.it`

Servono perché chi digita `memuraapp.it` senza `www` deve atterrare su
GitHub Pages, che farà 301 verso `www.memuraapp.it` (lo deduce dal CNAME).

```
Tipo: A | Nome: @ | Valore: 185.199.108.153 | TTL: 3600
Tipo: A | Nome: @ | Valore: 185.199.109.153 | TTL: 3600
Tipo: A | Nome: @ | Valore: 185.199.110.153 | TTL: 3600
Tipo: A | Nome: @ | Valore: 185.199.111.153 | TTL: 3600
```

(Questi sono gli IP ufficiali GH Pages per apex, validi da anni —
verifica comunque su https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain
se stai facendo il setup mesi dopo questo doc.)

### 4c. Eventuale conflitto con record `legal`

Il sottodominio `legal.memuraapp.it` ha **già** un CNAME esistente verso
`tech-memuraapp.github.io.` (configurato per `memura-legal`). Va lasciato
così — GitHub Pages gestisce più CNAME verso lo stesso host: distingue il
repo da servire in base al `CNAME` file dentro ciascun repo.

## 5. Verifica DNS + HTTPS

Dopo ~10-60 min di propagazione DNS:

```bash
dig www.memuraapp.it +short
# Atteso: tech-memuraapp.github.io. seguito da 4 IP 185.199.x.x

dig memuraapp.it +short
# Atteso: 4 IP 185.199.108-111.153
```

Su GitHub: Settings → Pages → vedrai
> Your site is published at https://www.memuraapp.it/
> ✓ DNS check successful.

Dopo che il check è verde, **spunta Enforce HTTPS**.

## 6. Test post-publish

| URL | Atteso |
|---|---|
| `https://www.memuraapp.it/` | landing |
| `http://www.memuraapp.it/` | 301 → HTTPS |
| `https://memuraapp.it/` | 301 → www |
| `https://www.memuraapp.it/?lang=en` | inglese |
| `https://www.memuraapp.it/robots.txt` | testo |
| `https://www.memuraapp.it/sitemap.xml` | XML |

## 7. Update workflow

Modifiche future:

```bash
# edit local files
git add . && git commit -m "fix: copy hero" && git push
# GitHub Pages re-builda in ~30s
```

---

## Asset PNG mancanti (generare prima del push o subito dopo)

```bash
# macOS, dopo `brew install librsvg`
rsvg-convert -w 1200 -h 630 assets/og-image.svg -o assets/og-image.png
rsvg-convert -w 180  -h 180  assets/favicon.svg  -o assets/apple-touch-icon.png

git add assets/og-image.png assets/apple-touch-icon.png
git commit -m "feat: og image and apple touch icon PNG"
git push
```

---

## TODO post-pubblicazione Play Store

- [ ] `index.html` → swap `href="#"` nei `.store-badge` con URL reali:
  - App Store: `https://apps.apple.com/it/app/memura/id<APP_ID>`
  - Play Store: `https://play.google.com/store/apps/details?id=com.memuraapp.memura`
- [ ] Google Play Console → **Negozio** → URL sito: `https://www.memuraapp.it/`.
- [ ] Email contatto sviluppatore: `tech@memuraapp.it`.
- [ ] Google Search Console: aggiungi proprietà `www.memuraapp.it` + sottometti `sitemap.xml`.

## Sviluppo locale

```bash
cd memura-landing
python3 -m http.server 8080
# Browser → http://localhost:8080
```
