# ⚡ FreeRam – Cloud & Webhosting Manuály

> Kompaktní návody na bezplatný cloud (Azure), hosting statických webů (Cloudflare Pages), správu DNS a CI/CD.  
> Styl: Pouze odrážky. Každé zbytečné slovo odebráno.

---

## 📚 Obsah repozitáře

### 1. [☁️ AZURE.md](./AZURE.md)
* **Student vs Standard:** Účet bez platební karty (100 $/rok jednorázově, ne měsíčně; B1s server nečerpá kredit) vs 30denní trial (200 $ kredit + 12 měsíců free).
* **B1s Server:** Konfigurace krok za krokem (Linux, SSH, 64 GB P6 SSD).
* **SWAP:** Nastavení 2 GB virtuální paměti pro 1 GB RAM.
* **Pasti na peníze:** Osiřelé disky, zapomenuté IP, limit 750 hodin, CPU throttling.
* **Záchranná brzda:** Nastavení Budget Alertu na 1 $ proti nechtěným platbám.

### 2. [🌐 WEBHOSTING.md](./WEBHOSTING.md)
* **Výběr hostingu:** Proč Cloudflare Pages drtí Render, Koyeb a Vercel (0 s cold start, Three.js rychlost).
* **Nasazení z GitHubu:** Konfigurace buildu pro Vite a React.
* **DNS (Český hosting):** 
  * CNAME pro `www` na Cloudflare.
  * A záznam na `91.239.200.50` s prázdnou subdoménou pro 301 přesměrování.
* **CI/CD & Náhledy:** Automatický deploy pro `main` + preview linky pro větve (např. `homepage`).
* **Backend & DB:** Pages Functions + bezplatná SQL databáze Cloudflare D1.
