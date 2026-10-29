# Webhosting & DNS – Manuál

> Vše o hostingu frontendu (Cloudflare Pages), DNS směrování (Český hosting), GitHub CI/CD a databázi.  
> Formát: Odrážky. Minimum slov.

---

## 1. Výběr hostingu pro Frontend (React / Vite / Three.js)

* **Render:**
  * Minus: Usíná po 15 min nečinnosti (cold start trvá 50 sekund).
  * Minus: Limit 750 hodin pro celý účet.
  * Minus: Zbytečný běh virtuálního kontejneru pro statické soubory.

* **Koyeb:**
  * Minus: Koupil ho Mistral AI → zablokovaný dashboard a onboarding pro běžné weby.

* **Vercel:**
  * Pozor: Často vyžaduje SMS verifikaci, blokuje některé české operátory.

* **Cloudflare Pages (Vítěz):**
  * Plus: 100% trvale zdarma, bez kreditní karty a bez SMS.
  * Plus: **Nikdy neusíná (0 s cold start).**
  * Plus: Celosvětová CDN (bleskové stahování 3D modelů a Three.js textur).
  * Plus: Neomezený počet návštěv a přenesených dat.
  * Plus: Automatický SSL certifikát (HTTPS).

---

## 2. Nasazení z GitHubu na Cloudflare Pages

1. V Cloudflare menu: **Compute** → **Workers & Pages**.
2. Záložka **Pages** → **Connect to Git**.
3. Vybrat repozitář (např. `MonsterMarian/Portfolio`).
4. **Build konfigurace:**
   * **Framework preset:** `React (Vite)`
   * **Build command:** `npm run build`
   * **Build output directory:** `dist`
   * **Root directory:** `/`
5. Kliknout na **Save and Deploy**.

---

## 3. Nastavení DNS (Cloudflare Authoritative DNS)

Cíl: Kompletní správa DNS a SSL pod Cloudflare pro obě verze domény (`marianvystavel.cz` i `www.marianvystavel.cz`).

* **1. Nameservery (u registrátora Český hosting):**
  * CZ.NIC NSSET nastaven na Cloudflare:
    * `adrian.ns.cloudflare.com`
    * `arnold.ns.cloudflare.com`
  * DNSSEC byl před změnou deaktivován.
  * Záznamy v administraci Českého hostingu jsou nyní irelevantní (veškeré DNS řídí Cloudflare).

* **2. Cloudflare Pages Custom Domains:**
  * `www.marianvystavel.cz` → CNAME na `portfolio-mo4.pages.dev` (Active • SSL).
  * `marianvystavel.cz` (apex) → CNAME `@` na `portfolio-mo4.pages.dev` přes CNAME Flattening (Active • SSL).

* **3. Výsledek:**
  * Obě adresy (s `www` i bez `www`) fungují okamžitě na celosvětové CDN.
  * Automatické, bezplatné obnovování SSL certifikátů.
  * Žádné chyby prošlých certifikátů z parking serverů.

---

## 4. GitHub CI/CD & Náhledové větve (Preview Deployments)

* **Produkční větev (`main`):**
  * Každý `git push origin main` automaticky spustí build na Cloudflare.
  * Změny jsou živě na `marianvystavel.cz` do 30 sekund.

* **Náhledové větve (Preview, např. `homepage`):**
  * Každý push do jiné větve vytvoří privátní testovací URL:
    * Formát: `https://<nazev-vetve>.<projekt>.pages.dev`
    * Příklad: `https://homepage.portfolio-mo4.pages.dev`
  * Umožňuje testovat grafiku a kód online bez zásahu do produkčního webu.

* **Spuštění preview buildu přes Git:**
  ```bash
  git checkout homepage
  git commit --allow-empty -m "chore: trigger preview build"
  git push origin homepage
  ```

---

## 5. Budoucí Backend & Databáze (Zero-cost řešení)

* **Zásada bezpečnosti:** Do databáze se nikdy nepřipojovat přímo z klientského Reactu (hesla by šla přečíst v DevTools prohlížeče).

* **Možnost 1: Cloudflare Pages Functions (Doporučeno):**
  * Do repozitáře přidat složku `functions/api/kontakt.js`.
  * Soubor se automaticky stane backendovým endpointem `/api/kontakt`.
  * Běží na stejné doméně (žádné CORS problémy).
  * 100 000 requestů denně zdarma.

* **Možnost 2: Databáze Cloudflare D1:**
  * Serverless SQL (SQLite) přímo v Cloudflare.
  * 5 GB databáze zdarma, 5 mil. čtení denně.
  * Přístup z Pages Function přes `context.env.DB`.

* **Možnost 3: Externí cloudové databáze zdarma:**
  * **Supabase:** PostgreSQL (500 MB zdarma, auth, storage).
  * **Neon.tech:** Serverless PostgreSQL.
  * **MongoDB Atlas:** 512 MB NoSQL cluster zdarma.
