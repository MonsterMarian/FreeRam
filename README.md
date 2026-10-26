# ⚡ Cloud & Web Deployment Cheatsheet

> Návod na bezplatný cloud (Azure), webhosting (Cloudflare Pages), DNS (Český hosting) a CI/CD z GitHubu.  
> Styl: Pouze odrážky. Žádná omáčka.

---

## 1. Azure: Standard vs Student

* **Azure for Students (Nejlepší volba, pokud máš školní mail):**
  * Odkaz: [azure.microsoft.com/free/students](https://azure.microsoft.com/free/students/)
  * **Kreditní karta:** NENÍ potřeba (0 Kč riziko).
  * **Kredit:** 100 USD na 12 měsíců.
  * **Obnova:** Každý rok studia znovu.
  * **Ověření:** Školní e-mail (.edu, univerzitní nebo ISIC).

* **Azure Free Trial (Standardní):**
  * Odkaz: [azure.microsoft.com/free](https://azure.microsoft.com/free)
  * **Kreditní karta:** Povinná (autorizační blokace ~1 USD / vrátí se).
  * **Kredit:** 200 USD na prvních 30 dní.
  * **12 měsíců free:** B1s server + 2× 64 GB P6 SSD + 100 GB egress.
  * **Záchranná brzda:** Po 30 dnech nutný ruční přechod na Pay-As-You-Go.

---

## 2. Azure B1s Server – Konfigurace krok za krokem

* **Resource Group:**
  * Vytvořit novou pro každý projekt (např. `rg-server-b1s`).
  * *Proč:* Smazáním Resource Group se smaže 100 % prostředků včetně disků a IP.

* **Basics:**
  * **Size:** Striktně `Standard_B1s` (1 vCPU, 1 GB RAM). Jediná 100% free velikost (750 h/měsíc).
  * **Region:** `Germany West Central` nebo `West Europe` (nejnižší ping).
  * **OS:** Ubuntu 24.04 LTS (Windows Server na 1 GB RAM extrémně swapuje a drhne).
  * **Auth:** SSH klíč (stáhnout `.pem`).

* **Disks (Kritické):**
  * **Typ:** Premium SSD.
  * **Velikost:** Ručně změnit na **64 GiB (P6)**.
  * *Pozor:* Azure defaultně předvolí 128 GiB (P10) – **128 GB JE PLACENÝCH!**
  * Zaškrtnout: ☑ *Delete OS disk with VM*.

* **Networking:**
  * **Inbound ports:** Povolit pouze port 22 (SSH).
  * Zaškrtnout: ☑ *Delete public IP and NIC when VM is deleted*.

* **Management (Vypnout placené pasti):**
  * ❌ *Azure Backup* -> Vypnuto.
  * ❌ *Microsoft Defender for Cloud* -> Vypnuto.
  * ❌ *Azure Bastion* -> Vypnuto (stojí 140 $/měsíc!).

* **Po startu Linuxu (Přidat Swap pro 1 GB RAM):**
  ```bash
  sudo fallocate -l 2G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  ```

---

## 3. Azure – Pasti na peníze & Checklist

* **Osiřelé disky (Orphaned Disks):**
  * Smazání VM nesmaže disk.
  * Zapomenutý disk stojí 15–20 $/měsíc.
  * *Obrana:* Mazat vždy celou Resource Group.

* **Neasociované IP adresy:**
  * Odpojená statická IP stojí ~3–4 $/měsíc.

* **Limit 750 hodin:**
  * 1 měsíc = 744 hodin.
  * Může běžet pouze **1 server B1s nonstop**.
  * Dva servery vyčerpají limit za 15 dní ➔ zbytek měsíce platíš.

* **Burstable CPU:**
  * B1s garantuje 10 % base CPU.
  * Trvalá zátěž vyčerpá kredity ➔ tvrdý throttling na 10 % ➔ server zamrzne na SSH.
  * Nevhodné pro herní servery a těžbu.

* **Záchranná pojistka (Budget Alert na 1 $):**
  * *Cost Management + Billing* ➔ *Budgets* ➔ *Add*.
  * Částka: **1 USD**.
  * Alert: 50 % a 100 %.
  * E-mail: tvůj mail.
  * Výsledek: Azure pošle varování při útratě 12 Kč dřív, než naskočí dluh.

---

## 4. Webhosting pro Frontend (React / Vite / Three.js)

* **Proč NE Render:**
  * Po 15 minutách usíná (cold start 50 s).
  * 750 h měsíční limit sdílený pro celý účet.
  * Zbytečný běh Node kontejneru pro čistě statické soubory.

* **Proč NE Koyeb:**
  * Prošel akvizicí pod Mistral AI ➔ zablokovaný dashboard pro běžné weby.

* **Proč Cloudflare Pages (Vítěz):**
  * 100% trvale zdarma, bez kreditní karty.
  * **Nikdy neusíná (0 s cold start).**
  * Celosvětová CDN (rychlé načítání těžkých Three.js 3D modelů).
  * Neomezený bandwidth.
  * SSL certifikát (HTTPS) zdarma.

* **Nastavení buildu v Cloudflare Pages:**
  * **Framework preset:** `React (Vite)`
  * **Build command:** `npm run build`
  * **Build output directory:** `dist`
  * **Root directory:** `/`

---

## 5. DNS Konfigurace (Český hosting / hosting.cz)

Cíl: Propojit `www.marianvystavel.cz` s Cloudflare a `marianvystavel.cz` přesměrovat na `www`.

* **Krok 1: CNAME pro `www` (Cloudflare Pages):**
  * Typ: `CNAME`
  * Subdoména: `www`
  * Cíl: `portfolio-mo4.pages.dev` (tvoje Pages URL).

* **Krok 2: Přesměrování webu (hosting.cz):**
  * Sekce *Alias, přesměrování* ➔ *Přesměrování webu*.
  * Cílová URL: `https://www.marianvystavel.cz`

* **Krok 3: A záznam pro doménu bez www (Český hosting redirect server):**
  * Typ: `A`
  * (Sub)doména: **nechat prázdné** (nevepisovat název domény!).
  * IP adresa: **`91.239.200.50`** (Český hosting server pro přesměrování parkovaných domén).
  * AAAA záznam: vytvoří se automaticky (`2001:67c:e94:0:1:5bef:c832:1`).

---

## 6. GitHub CI/CD & Náhledové větve (Preview Deployments)

* **Produkční větev (`main`):**
  * Každý `git push origin main` automaticky spustí build na Cloudflare.
  * Aktualizace na živé doméně do 30 sekund bez výpadku.

* **Vývojové větve (např. `homepage`):**
  * Push do jakékoliv jiné větve vytvoří privátní náhledový odkaz:
    * `https://<nazev-vetve>.portfolio-mo4.pages.dev`
    * Příklad: `https://homepage.portfolio-mo4.pages.dev`
  * Umožňuje testovat 3D grafiku a změny před mergem do produkce.

* **Vynucení sestavení náhledu přes Git:**
  ```bash
  git checkout <vetev>
  git commit --allow-empty -m "chore: trigger preview"
  git push origin <vetev>
  ```

---

## 7. Budoucí Backend & Databáze (Zero-cost architektura)

* **Zásada:** Nikdy nepřipojovat DB přímo z klientského Reactu (únik hesel v DevTools).

* **Možnost A: Cloudflare Pages Functions (Doporučeno):**
  * Složka `functions/api/kontakt.js` v repozitáři ➔ automatický backend endpoint `/api/kontakt`.
  * Žádný externí server, žádné CORS chyby.
  * 100 000 požadavků denně zdarma.

* **Možnost B: Cloudflare D1 Database:**
  * Serverless SQL (SQLite) přímo uvnitř Cloudflare.
  * 5 GB úložiště zdarma, 5 mil. čtení denně.
  * Žádná kreditní karta.

* **Možnost C: Externí databáze zdarma:**
  * **Supabase:** PostgreSQL (500 MB zdarma, auth, real-time).
  * **Neon.tech:** Serverless PostgreSQL.
  * **MongoDB Atlas:** 512 MB NoSQL cluster zdarma.
