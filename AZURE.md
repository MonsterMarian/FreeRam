# ☁️ Azure Free Tier – Linux server zdarma na 12 měsíců

> Ověřeno v praxi (10/2026, český účet). Jen odrážky, jen podstata.

---

## 1. Tarify

* **Azure for Students** – [azure.microsoft.com/free/students](https://azure.microsoft.com/free/students/)
  * Bez karty.
  * 100 USD **na rok** (ne měsíčně), obnova po re-verifikaci školního mailu.
  * Vyčerpání = služby se zastaví, nic se nestrhne.
* **Free Trial** – [azure.microsoft.com/free](https://azure.microsoft.com/free)
  * Karta povinná (jen ověření, 0 Kč).
  * 200 USD (~176 €) na 30 dní.
  * 12 měsíců zdarma: 750 h Linux VM, 2× 64 GB P6 SSD, 15 GB odchozích dat/měsíc.
  * ❌ Neklikat **Upgrade / Průběžné platby**.

![Kredit aktivní](img/azure/01-kredit.png)

---

## 2. Co je opravdu zdarma

* VM: **`Standard_B2ats_v2`** (x64, AMD) nebo **`Standard_B2pts_v2`** (ARM).
  * 2 vCPU, 1 GB RAM.
  * Ve výběru velikosti musí být štítek **„s nárokem na bezplatné služby“**.
* ❌ `B1s` – stará řada, už bez štítku.
* ❌ `B2ts_v2` – Azure ji nabízí při chybě, **ale je placená**.
* Disk: **64 GB Premium SSD (P6)**.

![Velikost se štítkem](img/azure/06-velikost.png)

---

## 3. Než začneš: registrace Microsoft.Compute

* Bez toho hlásí **`NotAvailableForSubscription`** všude.
* **Předplatná** ➔ *Azure subscription 1* ➔ **Poskytovatelé prostředků**.
* Najít `Microsoft.Compute` ➔ **Zaregistrovat** ➔ počkat na **Registered**.

![Microsoft.Compute NotRegistered](img/azure/05-compute-notregistered.png)

---

## 4. Region

* Většina EU regionů je pro trial **zablokovaná**:
  * North Europe, West Europe, Germany, France, UK, Norway, Poland, Switzerland, Italy.
* ✅ Funguje: **Sweden Central** (B2ats_v2 i B2pts_v2).
* Záloha: Spain Central, Austria East, Belgium Central, Denmark East (jen B2ats_v2).
* East US = zakázané pro EU účet.
* Seznam pro **tvůj** účet (Cloud Shell nebo Azure CLI):

```bash
az vm list-skus --all --resource-type virtualMachines \
  --query "[?(name=='Standard_B2ats_v2' || name=='Standard_B2pts_v2') && length(restrictions[?type=='Location'])==\`0\`].[locations[0], name]" -o table
```

![Region nepřijímá zákazníky](img/azure/07-region-zakazany.png)

---

## 5. Vytvoření VM – portál

* Přímý odkaz: [Bezplatné služby](https://portal.azure.com/#blade/Microsoft_Azure_Billing/FreeServicesBlade)
  * ❌ Ne přes „Virtuální počítače ➔ Vytvořit“ – tlačí placené řady D/E/F (případně **Přeskočit tento krok**).

![Bezplatné služby](img/azure/03-bezplatne-sluzby.png)
![Přeskočit tento krok](img/azure/02-preskocit.png)

* Dlaždice **Linuxový virtuální počítač** ➔ **Vytvořit**.
* Vyplnit:
  * Skupina prostředků: nová (1 projekt = 1 skupina).
  * Oblast: **Sweden Central**.
  * Image: Ubuntu 22.04 – **x64** pro B2ats_v2, **ARM64** pro B2pts_v2.
  * Velikost: se štítkem „s nárokem“.
  * Ověření: **SSH klíč** ➔ při vytvoření **stáhnout `.pem`** (jen jednou!).
  * Porty: jen **SSH (22)**.
* Disky průvodce nastaví sám (64 GB P6).

![Průvodce bezplatnými službami](img/azure/04-pruvodce.png)

---

## 6. Vytvoření VM – CLI (rychlejší, žádné klikání)

```bash
az group create -n rg-free -l swedencentral
az vm create -g rg-free -n srv-free -l swedencentral \
  --size Standard_B2pts_v2 \
  --image Canonical:0001-com-ubuntu-server-jammy:22_04-lts-arm64:latest \
  --admin-username azureuser --generate-ssh-keys \
  --os-disk-size-gb 64 --storage-sku Premium_LRS --security-type Standard
```

* x64 varianta: `--size Standard_B2ats_v2 --image Ubuntu2204`.
* Klíč se uloží do `~/.ssh/id_rsa` – zálohovat.

---

## 7. Chyba `AllocationFailed`

* = Microsoftu v regionu došel hardware. **Nastavení je OK.**
* Řešení:
  * Zkusit znovu později (`az vm start`).
  * Nebo druhá free velikost (x64 ⇄ ARM) – **ARM musí mít ARM64 image**.
  * Nebo jiný povolený region.
* ❌ Nebrat `B2ts_v2`, i když ji chyba doporučuje.

![AllocationFailed](img/azure/08-allocationfailed.png)

---

## 8. Hotovo – kontrola

* Stav: **Spuštěné**.
* Velikost: B2pts v2 / B2ats v2.
* Azure Spot: Zakázáno.
* Monitorování, zálohování: vypnuté.
* Připojení:

```bash
ssh -i cesta/ke/klici.pem azureuser@<veřejná-IP>
```

![Hotový server](img/azure/09-hotovo.png)

---

## 9. SWAP (nutné pro 1 GB RAM)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## 10. Pasti na peníze

* **Jen 1 VM nonstop** – 750 h = 1 server celý měsíc. Dva = půlka měsíce placená.
* **Vypínat v portálu** (tlačítko Zastavit = *deallocated*). `sudo poweroff` = hodiny běží dál.
* **Mazat celou skupinu prostředků** – smazání VM nechá disk a IP (placené).
* **Nepřipojená veřejná IP** = ~3–4 $/měsíc.
* **Placené doplňky** – Backup, Defender, Bastion (140 $/měsíc), Monitoring: nezapínat.
* **Burstable CPU** – nízký základní výkon, nad něj čerpá kredity; dlouhá zátěž = přiškrcení.
* **Po 30 dnech** – trial se zmrazí, VM se zastaví. Pro 12 měsíců zdarma nutný upgrade na průběžné platby (free kvóty platí dál).

---

## 11. Záchranná brzda: rozpočet 1 $

* **Cost Management + Billing** ➔ **Budgets** ➔ **+ Add**.
* Částka: **1**.
* Upozornění: 50 % a 100 %.
* Příjemce: svůj e-mail.
* Po pár dnech zkontrolovat **Cost Management ➔ Analýza nákladů** – musí být 0.
