# E-mail na vlastní doméně – Manuál

> Kompletní průvodce pro provoz e-mailu na vlastní doméně (`@marianvystavel.cz`) 100% zdarma.  
> Kombinace: **Cloudflare Email Routing** (příjem) + **Brevo SMTP** (odesílání).  
> Styl: Pouze odrážky. Žádný balast.

---

## 1. Jak to funguje (Architektura)

* **Příjem zpráv (Inbound):**
  * Někdo napíše na `marian@marianvystavel.cz` (nebo `kontakt@...`).
  * Cloudflare zachytí zprávu a okamžitě ji přepošle do tvé osobní schránky (Gmail, Seznam apod.).
  * Žádné diskové limity, nulové poplatky, neomezený počet aliasů.

* **Odesílání zpráv (Outbound):**
  * Pro odesílání se použije bezplatný SMTP účet (např. Brevo).
  * V Gmailu / Seznamu v nastavení zvolíš *„Odesílat poštu jako...“*.
  * Při psaní e-mailu si jen v roletce *Od:* vybereš svou doménovou adresu.
  * Zprávy mají platný digitální podpis (SPF + DKIM) a nepadají do spamu.

---

## 2. Krok 1: Nastavení příjmu (Cloudflare Email Routing)

### Kde to v Cloudflare najdeš:
1. V levém bočním menu klikni na **Domains** (ikonka zeměkoule).
2. Rozklikni doménu **`marianvystavel.cz`**.
3. V levém menu domény klikni na **Email** → **Email Routing**.
4. Klikni na **Get started** / **Enable Email Routing**.

### Konfigurace:
1. **Destination address (Kam má pošta chodit):**
   * Zadej svou běžnou soukromou schránku (např. tvůj `@gmail.com` nebo `@seznam.cz`).
   * Cloudflare ti na ni pošle ověřovací e-mail.
   * V e-mailu klikni na ověřovací odkaz (**Verify email address**).
2. **Custom address (Tvoje adresa na doméně):**
   * Zadej požadovaný tvar (např. `marian`, `kontakt`, nebo `info`).
   * Tím vznikne např. `marian@marianvystavel.cz`.
3. **Automatické DNS záznamy:**
   * Cloudflare ti nabídne automatické přidání potřebných MX a TXT záznamů do tvé DNS zóny.
   * Klikni na **Add records and enable**.
4. *(Volitelné)* **Catch-all adresa:**
   * V záložce *Routing rules* můžeš povolit *Catch-all* – jakýkoliv e-mail poslaný na cokoliv@marianvystavel.cz (i s překlepem) ti bezpečně dorazí.

---

## 3. Krok 2: Nastavení odesílání (Brevo SMTP – Zdarma)

Aby šlo z doménové adresy i odpovídat a psát nové zprávy:

1. **Registrace na Brevo.com:**
   * Vytvoř si bezplatný účet na [brevo.com](https://www.brevo.com) (tarif Free = 300 e-mailů/den zdarma).
2. **Ověření domény v Brevo:**
   * V Brevo jdi do: **Settings** → **Senders, Domains & Dedicated IPs** → **Domains** → **Add a domain**.
   * Zadej `marianvystavel.cz`.
   * Brevo ti vygeneruje DNS záznamy (TXT klíče pro DKIM).
3. **Vložení DKIM do Cloudflaru:**
   * V Cloudflare v detailu domény přejdi do **DNS** → **Records**.
   * Klikni na **+ Add record** a vlož TXT záznam podle Breva (hodnota `mail._domainkey`).
4. **Získání SMTP údajů:**
   * V Brevo klikni vpravo nahoře na svůj profil → **SMTP & API**.
   * Zkopíruj si:
     * **SMTP Server:** `smtp-relay.brevo.com`
     * **Port:** `587`
     * **Login:** tvůj e-mail u Breva
     * **Master Password / SMTP Key:** vygenerovaný klíč.

---

## 4. Krok 3: Propojení do tvého Gmailu (nebo jiné schránky)

1. V počítači otevři **Gmail** → klikni na ozubené kolečko (Nastavení) → **Zobrazit všechna nastavení**.
2. Přejdi na záložku **Účty a import**.
3. V sekci **Odesílat poštu jako:** klikni na **Přidat další e-mailovou adresu**.
4. Vyplň:
   * **Jméno:** Marian Vystavěl
   * **E-mailová adresa:** `marian@marianvystavel.cz`
   * Nech zaškrtnuto *Považovat za alias* → Další krok.
5. Zadej SMTP údaje z Breva:
   * **Server SMTP:** `smtp-relay.brevo.com`
   * **Port:** `587`
   * **Uživatelské jméno:** tvůj Brevo login
   * **Heslo:** tvůj Brevo SMTP klíč
   * Zabezpečené připojení pomocí TLS.
6. Klikni na **Přidat účet**.
7. Gmail ti pošle ověřovací kód (ten ti díky Cloudflare Email Routingu okamžitě přistane v Gmailu!). Kód opiš a potvrď.

---

## 5. Jak to používat v praxi

* **Při psaní nového e-mailu:**
  * V poli **Od:** klikneš na šipku a přepneš ze soukromého e-mailu na `marian@marianvystavel.cz`.
* **Při odpovědi na příchozí zprávu:**
  * Gmail umí automaticky odpovídat ze stejné adresy, na kterou zpráva přišla (lze zapnout v *Nastavení → Účty a import → Při odpovědi na zprávu: Odpovědět ze stejné adresy, na kterou byla zpráva odeslána*).
* **Test spolehlivosti (10/10 skóre):**
  * Na webu [mail-tester.com](https://www.mail-tester.com) získáš testovací adresu.
  * Pošli tam e-mail ze své nové adresy – zkontroluje správnost SPF, DKIM i to, zda nepadáš do spamu.
