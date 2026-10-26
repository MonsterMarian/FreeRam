# ☁️ Azure Free Tier – Manuál

> Vše o bezplatném cloudu Microsoft Azure: tarify, konfigurace serveru, pasti na peníze a bezpečnost.  
> Formát: Odrážky. Minimum slov.

---

## 1. Tarify: Student vs Standard

* **Azure for Students:**
  * Odkaz: [azure.microsoft.com/free/students](https://azure.microsoft.com/free/students/)
  * **Kreditní karta:** Není potřeba (nulové finanční riziko).
  * **Kredit:** **100 USD na ROK** (jednorázově na 12 měsíců, NE měsíčně).
  * **Čerpání kreditu:** B1s server běží zdarma z bezplatné kvóty (750 h/měsíc) – z kreditu 100 USD **neubírá nic**.
  * **Využití 100 USD:** Rezerva na silnější testovací stroje (např. 16 GB RAM na pár dní).
  * **Vyčerpání kreditu:** Služby se pouze zastaví. Nikdy se nic nestrhne.
  * **Obnova:** Každých 12 měsíců re-verifikace školního mailu ➔ načte se dalších 100 USD.
  * **Podmínka:** Školní e-mail (.edu, univerzitní nebo ISIC).

* **Azure Free Trial:**
  * Odkaz: [azure.microsoft.com/free](https://azure.microsoft.com/free)
  * **Karta:** Povinná (testovací blokace ~1 USD / ihned se uvolní).
  * **Kredit:** 200 USD na prvních 30 dní.
  * **12 měsíců zdarma:** B1s server + 2× 64 GB P6 SSD + 100 GB odchozích dat/měsíc.
  * **Konec 30 dní:** Subscription se zmrazí ➔ nutný ruční upgrade na Pay-As-You-Go.

---

## 2. Konfigurace B1s serveru (Krok za krokem)

* **Resource Group:**
  * Vytvořit novou (např. `rg-server-b1s`).
  * *Pravidlo:* 1 projekt = 1 Resource Group.
  * *Důvod:* Smazáním Resource Group se smaže vše (žádné zapomenuté placené disky).

* **Basics:**
  * **Size:** Striktně `Standard_B1s` (1 vCPU, 1 GB RAM). Jediná 100% free velikost.
  * **Region:** `Germany West Central` nebo `West Europe` (nejnižší ping).
  * **OS:** Ubuntu 24.04 LTS (Windows Server na 1 GB RAM extrémně drhne).
  * **Auth:** SSH klíč (stáhnout soubor `.pem`).

* **Disks (Kritický krok):**
  * **Typ:** Premium SSD.
  * **Velikost:** Změnit na **64 GiB (P6)**.
  * *Pozor:* Azure předvolí 128 GiB (P10) – **128 GB je placených!**
  * Zaškrtnout: ☑ *Delete OS disk with VM*.

* **Networking:**
  * **Inbound ports:** Povolit pouze port 22 (SSH).
  * Zaškrtnout: ☑ *Delete public IP and NIC when VM is deleted*.

* **Management (Vypnout skryté poplatky):**
  * ❌ *Azure Backup* -> Vypnout (placené).
  * ❌ *Microsoft Defender for Cloud* -> Vypnout (placené po trialu).
  * ❌ *Azure Bastion* -> Vypnout (stojí 140 $/měsíc!).

---

## 3. Nastavení SWAPu (Nutné pro 1 GB RAM)

Bez swapu systém ukončuje procesy (OOM killer):
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

---

## 4. Pasti na peníze & Technické limity

* **Zapomenuté disky (Orphaned Disks):**
  * Smazání virtuálky nesmaže její disk.
  * Samotný disk stojí 15–20 $/měsíc.
  * *Obrana:* Vždy mazat celou Resource Group.

* **Neasociované IP adresy:**
  * Odpojená veřejná IPv4 adresa stojí ~3–4 $/měsíc.
  * *Obrana:* Zkontrolovat v portálu sekci *Public IP addresses* (žádná nesmí být ve stavu *Unassociated*).

* **Limit 750 hodin:**
  * 1 měsíc = 744 hodin.
  * Zdarma je běh pouze **1 serveru B1s nonstop**.
  * Dva servery současně vyčerpají limit za 15 dní ➔ druhá polovina měsíce je placená.

* **Vypnutí serveru (Stopped vs Deallocated):**
  * `sudo poweroff` v Linuxu = stav *Stopped* (hardware je stále rezervovaný, hodiny se počítají!).
  * Správné vypnutí = tlačítko **Stop** v Azure portálu (stav *Stopped (deallocated)*).

* **Burstable CPU:**
  * B1s má základní výkon 10 % CPU.
  * Zátěž nad 10 % čerpá kredity.
  * Po vyčerpání kreditů dojde k přiškrcení na 10 % ➔ server přestane reagovat na SSH.
  * Nevhodné pro herní servery a trvalé výpočty.

---

## 5. Záchranná brzda: Budget Alert na 1 $

Nastavit ihned po registraci:
1. Horní lišta ➔ vyhledat **Cost Management + Billing**.
2. Vlevo: **Cost Management** ➔ **Budgets** ➔ **+ Add**.
3. **Budget amount:** Zadat **1** (1 USD / ~25 Kč).
4. **Alert conditions:**
   * 50 % (upozornění při útratě 0.50 $).
   * 100 % (upozornění při útratě 1.00 $).
5. **Recipients:** Zadat vlastní e-mail.
6. Výsledek: Okamžitý e-mail při jakémkoliv nechtěném výdaji.
