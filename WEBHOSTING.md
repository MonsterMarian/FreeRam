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

## 3. Nastavení DNS (Český hosting / hosting.cz)

Cíl: Propojit `www.marianvystavel.cz` s Cloudflare a `marianvystavel.cz` (bez www) přesměrovat na `www`.

* **1. Propojení subdomény `www` (v Cloudflare):**
  * Projekt → *Custom domains* → *Set up a custom domain*.
  * Zadat `www.marianvystavel.cz`.
  * Zvolit **My DNS provider** → **Begin CNAME setup**.

* **2. Nastavení CNAME záznamu (na hosting.cz):**
  * Sekce **CNAME záznamy** → tužka u `www.marianvystavel.cz`.
  * Typ: `CNAME`
  * Cíl: `portfolio-mo4.pages.dev` (tvoje Pages URL).
  * Uložit.

* **3. Nastavení přesměrování pro verzi bez www (na hosting.cz):**
  * Sekce **Alias, přesměrování** → **Přesměrování webu**.
  * Cílová URL: `https://www.marianvystavel.cz`
  * Typ: 301 trvalé.

* **4. Nastavení A záznamu pro doménu bez www (na hosting.cz):**
  * Sekce **A/AAAA záznamy** → zelené tlačítko **+ Přidat A záznam**.
  * **(Sub)doména:** Nechat **ÚPLNĚ PRÁZDNÉ** (nevepisovat název domény, jinak vznikne duplikát!).
  * **IP adresa:** **`91.239.200.50`** (server Českého hostingu pro obsluhu přesměrování).
  * AAAA záznam doplní hosting automaticky (`2001:67c:e94:0:1:5bef:c832:1`).

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
