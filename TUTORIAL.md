# 🚀 Microsoft Azure Free Tier – Kompletní tutoriál krok za krokem

Tento návod tě provede bezpečným založením a provozem virtuálního serveru (VM) na Microsoft Azure, abys vytěžil maximum z **12 měsíců zdarma** a počátečního kreditu **200 USD na prvních 30 dní**, aniž bys zaplatil jedinou korunu.

---

## 📋 Co přesně je zdarma (Free Tier přehled)

1. **Počáteční kredit (prvních 30 dní):**
   - **200 USD** (cca 4 500 Kč) kredit na vyzkoušení jakýchkoliv Azure služeb.
2. **Na 12 měsíců zdarma (měsíčně se obnovuje):**
   - **750 hodin** virtuálního stroje velikosti **B1s** (1 vCPU, 1 GB RAM, burstable architektura). 750 hodin pokryje nepřetržitý běh **jednoho** serveru měsíčně (31 dní × 24 h = 744 h).
   - **2× 64 GB disky (P6 Premium SSD)** nebo Standard SSD/HDD.
   - **100 GB** odchozího síťového provozu měsíčně (egress).
3. **Trvale zdarma (Always Free):**
   - Menší doplňkové služby (např. Azure App Service F1, Azure Cosmos DB malá kvóta, Azure Functions 1 mil. volání měsíčně).

---

## 🛠️ Fáze 1: Registrace a založení účtu

1. Jdi na [azure.microsoft.com/free](https://azure.microsoft.com/free) a klikni na **Začít zdarma** (*Start free*).
2. Přihlas se pomocí Microsoft účtu (nebo si vytvoř nový).
3. **Ověření totožnosti:**
   - Zadej telefonní číslo pro SMS kód.
   - Zadej platební kartu. **Důležité:** Karta slouží pouze k ověření identity. Microsoft provede dočasnou blokaci cca 1 USD / 25 Kč (která se ihned uvolní). Během prvních 30 dní je aktivní tzv. *Spending Limit* nastavený na 0 USD – nic se nestrhne.

---

## 🛡️ Fáze 2: Klíčová strategie Resource Groups (Skupiny prostředků)

> [!IMPORTANT]
> **Zlaté pravidlo Azure:** Nikdy nevytvářej servery a jejich disky do jedné společné hromady.
> Když smažeš pouze virtuální mašinu (VM), Azure v systému **ponechá disk a veřejnou IP adresu**, za které ti po 30 dnech začne účtovat peníze!
> 
> Řešení: **Pro každý server vytvoř samostatnou Resource Group.** Když pak smažeš celou Resource Group, smaže se 100 % prostředků (včetně disků, síťovek a IP) bez jakýchkoliv skrytých zbytků.

Vytvoříme si dvě skupiny:
- `rg-celorocni-b1s`: Pro trvalý server na 12 měsíců.
- `rg-testovaci-kredit`: Pro výkonné stroje, které budeš testovat za 200 USD kredit a před koncem 30 dní celou skupinu smažeš.

---

## 🖥️ Fáze 3: Vytvoření ročního bezplatného serveru (B1s)

1. V Azure Portálu ([portal.azure.com](https://portal.azure.com)) vyhledej nahoře **Virtual machines** a klikni na **Create** -> **Azure virtual machine**.
2. **Karta Basics (Základní údaje):**
   - **Subscription:** *Azure subscription 1* (nebo *Free Trial*).
   - **Resource group:** Klikni na *Create new* a zadej např. `rg-celorocni-b1s`.
   - **Virtual machine name:** např. `srv-free-b1s`.
   - **Region:** Vyber evropský region s dobrou odezvou, např. `(Europe) Germany West Central` nebo `(Europe) West Europe`.
   - **Availability options:** *No infrastructure redundancy required*.
   - **Security type:** *Standard* (nebo Trusted launch).
   - **Image:** Vyber požadovaný OS (např. *Ubuntu Server 24.04 LTS x64* nebo *Debian 12*). Pokud chceš Windows Server, zkontroluj, zda je dostupný v B1s. Pro 1 GB RAM je Linux nesrovnatelně svižnější.
   - **Size (Velikost):** Klikni na *See all sizes* a vyhledej **Standard_B1s** (1 vcpu, 1 GiB memory). U této velikosti je často přímo štítek *"Free services eligible"*.
3. **Správa přístupu (Administrator account):**
   - **Authentication type:** Doporučeno **SSH public key**.
   - **Username:** např. `azureuser`.
   - **SSH key source:** *Generate new key pair* (při vytváření se ti stáhne `.pem` soubor do PC – ten si dobře ulož!).
4. **Karta Disks (Extrémně důležité):**
   - **OS disk type:** Vyber **Premium SSD (locally-redundant storage)**.
   - **OS disk size:** Zvol **Change size** a vyber **64 GiB (P6)**. Defaultně Azure často předvybere 128 GiB (P10) – **128 GB disk NENÍ ve Free Tieru a po 30 dnech by byl placený!** Musíš ručně zvolit **P6 (64 GB)**.
   - **Delete disk with VM:** Zaškrtni políčko ☑ *Delete OS disk with VM* (pro jistotu).
5. **Karta Networking:**
   - **Virtual network & Subnet:** Ponech automaticky vytvořené v tvé Resource Group.
   - **Public IP:** Nech vytvořit novou (bude přiřazená k VM).
   - **NIC network security group:** *Basic*.
   - **Public inbound ports:** Povol *Allow selected ports* a vyber **SSH (22)** pro Linux (nebo RDP 3389 pro Windows).
   - **Delete public IP and NIC when VM is deleted:** Zaškrtni políčko ☑.
6. **Karta Management & Monitoring (POZOR NA POPLATKY):**
   - **Microsoft Defender for Cloud:** Vypnout / Free.
   - **Identity:** *Off*.
   - **Enable auto-shutdown:** Můžeš nastavit, pokud chceš, ale B1s může běžet 24/7 (744 h za měsíc je pod limitem 750 h).
   - **Backup (Zálohování):** Ujisti se, že je **vypnuté** (Azure Backup je placená služba!).
7. **Review + create:**
   - Zkontroluj shrnutí a klikni na **Create**.
   - Stáhni privátní klíč (`srv-free-b1s_key.pem`).

---

## 🔑 Fáze 4: Připojení k serveru přes SSH (z Windows)

1. Otevři PowerShell nebo Terminál v adresáři, kde máš stažený `.pem` klíč.
2. Ve Windows nastav práva souboru (pokud by SSH hlásilo "permissions are too open"):
   ```powershell
   icacls .\srv-free-b1s_key.pem /inheritance:r
   icacls .\srv-free-b1s_key.pem /grant:r "$($env:USERNAME):(R)"
   ```
3. Připoj se k veřejné IP adrese (najdeš ji v Azure na detailu virtuálního stroje):
   ```bash
   ssh -i .\srv-free-b1s_key.pem azureuser@<VEREJNA_IP_ADRESA>
   ```

---

## ⚡ Fáze 5: Nastavení SWAP paměti (Klíčové pro 1 GB RAM)

1 GB RAM u B1s se zaplní velmi rychle. Nastavení 2–4 GB Swapu zabrání pádu systému (OOM killer):

```bash
# Vytvoření 2GB swap souboru
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Trvalé zachování po restartu
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimalizace využití swapu
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

---

## 🧪 Fáze 6: Jak využít 200 USD kredit na zkoušku (prvních 30 dní)

Chceš si vyzkoušet silný server s 16 GB RAM, GPU nebo více nodů?
1. Založ novou Resource Group: `rg-testovaci-kredit`.
2. Do ní vytvoř jakékoliv výkonné mašiny.
3. Kredity se odečítají z 200 USD daru.
4. **Kritický krok:** **28. nebo 29. den** jdi do *Resource groups* -> otevři `rg-testovaci-kredit` -> klikni nahoře na **Delete resource group** a potvrď zadáním jejího názvu.
   - Tím je zaručeno, že v účtu nezůstane jediný zapomenutý placený disk, IP ani síťovka!

---

## 🔄 Fáze 7: Přechod po 30 dnech (Upgrade na Pay-As-You-Go)

Po 30 dnech (nebo po vyčerpání 200 USD) Azure zmrazí bezplatný účet (*Disabled*).
Aby ti B1s server běžel zbývajících 11 měsíců:
1. Ujisti se, že jsi smazal všechny testovací Resource Groups a zůstala jen `rg-celorocni-b1s`.
2. V Azure Portálu klikni na banner **Upgrade subscription** (přechod na Pay-As-You-Go).
3. **Nebudeš nic platit**, pokud dodržíš limity Free Tieru popsané v souboru [NA_CO_SI_DAT_POZOR.md](file:///c:/Users/mvystavel/Grav-Projekty/FreeRamXD/NA_CO_SI_DAT_POZOR.md).
