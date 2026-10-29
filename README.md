# FreeRam – Cloud & Webhosting Manuály

> Kompaktní návody na bezplatný cloud (Azure), hosting statických webů (Cloudflare Pages), správu DNS a CI/CD.  
> Styl: Pouze odrážky. Každé zbytečné slovo odebráno.

---

## Obsah repozitáře

### 1. [AZURE.md](./AZURE.md)
* **Student vs Standard:** Účet bez platební karty (100 $/rok) vs 30denní trial (200 $ kredit + 12 měsíců free).
* **Co získáš:** 2 vCPU, 1 GB RAM, 2× 64 GB SSD, 15 GB přenosu, DB a další služby na 12 měsíců.
* **Free VM:** `B2ats_v2` / `B2pts_v2` v **Sweden Central** – portál i CLI, se screenshoty.
* **Chyby:** `NotAvailableForSubscription` (Microsoft.Compute, zakázané regiony), `AllocationFailed`.
* **SWAP:** Nastavení 2 GB virtuální paměti pro 1 GB RAM.
* **Pasti na peníze:** Osiřelé disky, zapomenuté IP, limit 750 hodin, placená `B2ts_v2`.
* **Záchranná brzda:** Nastavení Budget Alertu na 1 $ proti nechtěným platbám.

### 2. [WEBHOSTING.md](./WEBHOSTING.md)
* **Výběr hostingu:** Proč Cloudflare Pages drtí Render, Koyeb a Vercel (0 s cold start, Three.js rychlost).
* **Nasazení z GitHubu:** Konfigurace buildu pro Vite a React.
* **DNS (Cloudflare Authoritative DNS):** 
  * Delegace nameserverů na Cloudflare.
  * CNAME pro `www` i apex (`@`) přes CNAME Flattening.
* **CI/CD & Náhledy:** Automatický deploy pro `main` + preview linky pro větve (např. `homepage`).
* **Backend & DB:** Pages Functions + bezplatná SQL databáze Cloudflare D1.

### 3. [EMAIL.md](./EMAIL.md)
* **Architektura:** Cloudflare Email Routing (příjem) + Brevo SMTP (odesílání).
* **Příjem zdarma:** Nastavení forwarding pravidel a cílové schránky v Cloudflare.
* **Odesílání z domény:** Brevo účet (300 mailů/den free) + SPF/DKIM podpisy proti spamu.
* **Propojení do schránky:** Nastavení „Odesílat poštu jako“ v Gmailu.
