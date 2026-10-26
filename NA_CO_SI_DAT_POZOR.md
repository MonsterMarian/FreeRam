# ⚠️ Azure Free Tier – Velký průvodce: Na co si dát pozor a pasti na peníze

Tento dokument slouží jako **kompletní bezpečnostní manuál** pro provoz Microsoft Azure v bezplatném režimu. Azure je enterprise cloud – je navržen tak, aby podnikům účtoval každou minutu a každý gigabyte. Pokud nedodržíš přesná pravidla, začne ti po přechodu na Pay-As-You-Go automaticky strhávat peníze z karty.

Zde je detailní rozbor všech pastí, skrytých nákladů a technických limitů.

---

## 📑 Rychlý rozcestník

1. [Past #1: Zbytkové disky po smazání virtuálky (Orphaned Disks)](#1-past-1-zbytkové-disky-po-smazání-virtuálky-orphaned-disks)
2. [Past #2: Neasociované a vyhrazené veřejné IP adresy](#2-past-2-neasociované-a-vyhrazené-veřejné-ip-adresy)
3. [Past #3: Limit 750 hodin a souběh serverů](#3-past-3-limit-750-hodin-a-souběh-serverů)
4. [Past #4: Výchozí velikost disku (128 GB vs 64 GB)](#4-past-4-výchozí-velikost-disku-128-gb-vs-64-gb)
5. [Past #5: Burstable CPU – Jak funguje řada B1s a proč zamrzne](#5-past-5-burstable-cpu--jak-funguje-řada-b1s-a-proč-zamrzne)
6. [Past #6: Přechod po 30 dnech a ztráta „záchranné brzdy“](#6-past-6-přechod-po-30-dnech-a-ztráta-záchranné-brzdy)
7. [Past #7: Skryté doplňkové služby (Backup, Defender, Bastion, Monitor)](#7-past-7-skryté-doplňkové-služby-backup-defender-bastion-monitor)
8. [Past #8: Síťový provoz (Egress) a přenos dat mezi regiony](#8-past-8-síťový-provoz-egress-a-přenos-dat-mezi-regiony)
9. [Past #9: Zastavení serveru („Stopped“ vs „Deallocated“)](#9-past-9-zastavení-serveru-stopped-vs-deallocated)
10. [Past #10: Konec 12měsíční lhůty](#10-past-10-konec-12měsíční-lhůty)
11. [🛡️ Záchranná brzda: Nastavení Budget Alertu na 1 $](#-záchranná-brzda-nastavení-budget-alertu-na-1-)
12. [✅ Kontrolní checklist (Před upgradem na Pay-As-You-Go)](#-kontrolní-checklist-před-upgradem-na-pay-as-you-go)

---

## 1. Past #1: Zbytkové disky po smazání virtuálky (Orphaned Disks)

> [!CAUTION]
> **Nejčastější důvod nechtěných plateb v Azure!**

Když v Azure portálu klikneš pravým tlačítkem na virtuální server a zvolíš **Delete**, ve výchozím nastavení se smaže **pouze výpočetní instance** (procesor a RAM).
Virtuální disk (OS disk nebo Data disk) **v systému zůstane existovat** jako samostatný zdroj.

- **Proč je to problém:**
  - Ve Free Tieru máš zdarma pouze **dva 64GB disky (P6)**.
  - Pokud jsi testoval výkonný server s 128GB diskem (P10) a server smažeš, 128GB disk ti v účtu zůstane.
  - V momentě, kdy po 30 dnech přejdeš na Pay-As-You-Go, ti Azure za tento zapomenutý disk začne strhávat cca **15–20 USD měsíčně**, i když server vůbec neběží!

### Jak se tomu vyhnout:
1. **Používej samostatné Resource Groups:** Když chceš cokoliv testovacího smazat, **smaž celou Resource Group** (*Delete resource group*). To zaručí 100% vymazání disku, síťové karty i IP adresy.
2. **Při vytváření zaškrtni:** V kartě *Disks* vždy zaškrtni políčko `Delete OS disk with VM`.
3. **Pravidelná kontrola:** Ve vyhledávání Azure zadej **Disks** a zkontroluj, zda u žádného disku není ve sloupci *Managed by* prázdno (prázdno = disk není nikde připojený, ale platíš za něj).

---

## 2. Past #2: Neasociované a vyhrazené veřejné IP adresy

Azure účtuje poplatky za **veřejné IP adresy (Public IPs)** podle specifických pravidel:

- **Dynamická / standardní IP přiřazená k běžící VM:** V rámci Free balíčku je kryta.
- **Odpojená veřejná IP (Unassociated IP):** Pokud smažeš VM, ale nesmažeš její veřejnou IP adresu, Azure ti ji začne účtovat, protože pro tebe drží IPv4 adresu na svých routerech (cca 3–4 USD měsíčně).
- **Více veřejných IP:** Každá další IP adresa nad rámec základní je placená.

### Jak se tomu vyhnout:
- V kartě *Networking* při vytváření VM vždy zaškrtni `Delete public IP and NIC when VM is deleted`.
- Ve vyhledávání zadej **Public IP addresses** a ověř, že neexistuje žádná IP se stavem *Not associated*.

---

## 3. Past #3: Limit 750 hodin a souběh serverů

Free Tier ti dává **750 hodin měsíčně** pro velikost **B1s**.

- Měsíc s 31 dny má: `31 × 24 = 744 hodin`.
- To znamená, že **jeden server** může běžet 24 hodin denně, 7 dní v týdnu po celý měsíc a stále ti zbývá rezerva 6 hodin.

> [!WARNING]
> Pokud ale vytvoříš **dva B1s servery současně**:
> - Každou hodinu reálného času spotřebuješ 2 hodiny z limitu.
> - Limit 750 hodin vyčerpáš za **15,5 dne**!
> - Od 16. dne do konce měsíce platíš za oba servery plnou komerční taxu!

### Pravidlo:
Měj spuštěný **maximálně JEDEN B1s server současně**. Pokud potřebuješ vytvořit jiný, ten první musíš zastavit (deallocate).

---

## 4. Past #4: Výchozí velikost disku (128 GB vs 64 GB)

Při vytváření virtuálního stroje v portálu Azure ti průvodce velmi často automaticky předvybere:
- `Premium SSD 128 GiB (P10)` nebo `30 GiB`.

Free Tier specifikace pro 12 měsíců však říká:
- **2× 64 GiB (P6) Premium SSD** je zdarma.

Pokud necháš předvolený **128 GB disk (P10)**, nespadá do bezplatného limitu 64 GB a po vypršení 30denního kreditu se ti začne účtovat plná cena (cca 18 USD měsíčně)!

### Jak správně nastavit:
Při vytváření VM v záložce **Disks** klikni na **Change size** a vyber ručně **P6 – 64 GiB**.

---

## 5. Past #5: Burstable CPU – Jak funguje řada B1s a proč zamrzne

Velikost `Standard_B1s` má specifickou hardwarovou architekturu:
- **1 vCPU, 1 GB RAM**
- Písmeno **B** znamená **Burstable**.
- Garantovaný základní výkon je pouze **10 % kapacity procesoru**!

### Jak funguje kreditní systém:
- Když je procesor vytížen pod 10 %, server si „střádá“ CPU kredity.
- Když potřebuješ nárazový výkon (např. instalace balíčků, start aplikace), server může vyletět na 100 % CPU tím, že spotřebovává naspořené kredity.
- **Když kredity dojdou:** Server je tvrdě přiškrcen (throttled) na **10 % CPU**.

### Důsledky pro praxi:
- Pokud na B1s spustíš něco s trvalým vytížením (např. Minecraft server, náročný bot, těžení kryptoměn, nekonečný cyklus ve skriptu):
  1. Během 30–60 minut vyčerpáš všechny CPU kredity.
  2. Server se stane extrémně pomalým.
  3. Přestane reagovat na SSH připojení a může se jevit jako mrtvý.
- **Řešení:** B1s je určen na nenáročné webové stránky, API backendy, cron skripty a testování. Není určen na trvalou výpočetní zátěž.

---

## 6. Past #6: Přechod po 30 dnech a ztráta „záchranné brzdy“

Během prvních 30 dní má tvůj účet stav **Free Trial**.
- V tomto stavu je aktivní **Spending Limit (limit útrat) = 0 USD**.
- Pokud bys udělal chybu, Azure ti prostě služby vypne, ale **nemůže ti strhnout ani korunu**.

### Co se stane 30. den:
1. Kredit 200 USD propadne (i kdybys ho nevyčerpal).
2. Subscription přejde do stavu **Disabled** (všechny servery se zastaví).
3. Aby ti B1s server běžel dalších 11 měsíců, musíš kliknout na **Upgrade na Pay-As-You-Go**.
4. **V TOMTO MOMENTĚ ZTRÁCÍŠ ZÁCHRANNÝ LIMIT!**
   - Od této chvíle se jakékoliv překročení bezplatných kvót ihned strhává z tvé platební karty.

---

## 7. Past #7: Skryté doplňkové služby (Backup, Defender, Bastion, Monitor)

Při proklikávání tvorby VM Azure nabízí řadu „doporučených“ služeb, které jsou ve výchozím stavu buď zaškrtnuté, nebo tě k nim navádí:

| Služba | Co to je | Proč se vyhnout ve Free Tieru |
| :--- | :--- | :--- |
| **Azure Backup** | Zálohování celého disku VM | Placená služba (úložiště + instance licence, cca 10–15 $/měsíc). |
| **Microsoft Defender for Cloud** | Bezpečnostní audit a ochrana | Po 30denní zkušební době stojí cca 15 $ za server měsíčně! |
| **Azure Bastion** | Připojení přes browser bez veřejné IP | Stojí cca **140 USD měsíčně** (naprostá finanční past pro začátečníky!). |
| **Azure Monitor / Log Analytics** | Sbírání telemetrie a logů | Nad 5 GB dat měsíčně je placené. |
| **Storage Account Diagnostics** | Ukládání boot diagnostiky | Vždy zvol *Enable with managed storage account (recommended)* – to je zdarma. Nevytvářej vlastní placený Storage Account pro diagnostiku. |

---

## 8. Past #8: Síťový provoz (Egress) a přenos dat mezi regiony

- **Příchozí data (Ingress):** Jsou v Azure **vždy zdarma** (nahrávání dat na server).
- **Odchozí data (Egress):** Máš zdarma **100 GB měsíčně** z celého Azure do internetu.
  - Pokud ze serveru budeš streamovat video nebo hostovat velké soubory a překročíš 100 GB, platí se cca 0.087 USD za každý další GB.
- **Meziregionální provoz:** Pokud máš VM v regionu *West Europe* a databázi nebo disk v *North Europe*, platíš za přenos dat mezi regiony i uvnitř sítě Azure. Měj všechny prostředky vždy v **jednom stejném regionu**.

---

## 9. Past #9: Zastavení serveru („Stopped“ vs „Deallocated“)

Pokud server nepotřebuješ a chceš, aby nečerpal hodiny z limitu 750 h:

- **Chyba – Vypnutí uvnitř OS:** Pokud dáš v Linuxu `sudo poweroff`, server přejde do stavu **Stopped**.
  - V tomto stavu Azure pro tebe **stále rezervuje hardware** a hodiny se dál započítávají!
- **Správně – Vypnutí přes Azure Portal:** V portálu klikni na tlačítko **Stop**.
  - Server musí mít stav **Stopped (deallocated)**.
  - Teprve v tomto stavu se neuplatňuje čerpání 750 hodin procesoru.
  - *(Pozor: Za disky v účtu se "virtuální nájem" počítá stále, ale v případě 64GB disku je pokryt Free kvótou).*

---

## 10. Past #10: Konec 12měsíční lhůty

Bezplatné výhody (B1s a 64GB SSD) platí přesně **12 měsíců od data registrace**.

- Microsoft ti nepošle žádné červené varování, které by server samo vypnulo.
- 13. měsíc začne Azure B1s server a disk účtovat jako standardní Pay-As-You-Go produkt (cca 10–12 USD měsíčně za server + disk).
- **Doporučení:** Nastav si hned při registraci upomínku do kalendáře na **11. měsíc a 25. den**, abys data zazálohoval a celou Resource Group smazal, pokud už nechceš platit.

---

## 🛡️ Záchranná brzda: Nastavení Budget Alertu na 1 $

Toto je **nejdůležitější krok**, který musíš udělat ihned po založení účtu. Nastavíme automatický e-mailový poplach, pokud by se na účtu objevila jakákoliv útrata (např. 0.50 $ nebo 1 $).

### Postup nastavení:
1. V horním vyhledávacím řádku Azure zadej **Cost Management + Billing**.
2. V levém menu klikni na **Cost Management** -> **Budgets**.
3. Klikni na **+ Add** (Přidat rozpočet).
4. Nastav parametry:
   - **Scope:** Tvoje Subscription.
   - **Name:** `budget-strazce-nuly`.
   - **Reset period:** *Monthly* (Měsíčně).
   - **Creation date / Expiration date:** Nech výchozí.
   - **Budget amount:** Zadej **1** (tj. 1 USD nebo odpovídající částku v CZK, např. 25 Kč).
5. Klikni na **Next: Set alerts**:
   - **Alert condition:** *Actual* (Skutečná útrata).
   - **% of budget:** Zadej `50` (upozornění přijde při útratě 0.50 $) a přidej další řádek se `100` (při útratě 1 $).
   - **Email recipients:** Zadej svůj e-mail, který aktivně čteš.
6. Klikni na **Create**.

> [!TIP]
> Jakmile by ti v budoucnu utekl zapomenutý disk, IP nebo překročený limit, Azure ti během pár hodin pošle varovný e-mail a ty můžeš problém ihned vyřešit, ještě než naskočí vyšší částka.

---

## ✅ Kontrolní checklist (Před upgradem na Pay-As-You-Go)

Předtím, než po 30 dnech klikneš na tlačítko *Upgrade to Pay-As-You-Go*, projdi si tento kontrolní seznam:

- [ ] **Smazal jsi testovací stroje:** Celá Resource Group `rg-testovaci-kredit` byla kompletně smazána.
- [ ] **Ověření disků:** V sekci *Disks* vidíš pouze **jeden disk** o velikosti **64 GiB (P6)**. Žádné 128 GB ani 256 GB disky.
- [ ] **Žádné osiřelé disky:** Sloupec *Owner / Managed by* u disku obsahuje název tvé B1s virtuálky (není prázdný).
- [ ] **Počet VM:** V sekci *Virtual machines* máš pouze **1 běžící stroj** velikosti `Standard_B1s`.
- [ ] **Ověření IP adres:** V sekci *Public IP addresses* máš pouze 1 IP adresu, která je přiřazena k běžícímu serveru.
- [ ] **Zálohy a doplňky:** U virtuálky je vypnutý *Azure Backup* a *Microsoft Defender for Cloud*.
- [ ] **Budget Alert:** V *Cost Management* máš aktivní rozpočet nastavený na 1 USD s tvým e-mailem.
- [ ] **Kalendář:** Máš nastavenou upomínku na konec 12měsíční bezplatné lhůty.
