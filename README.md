
# Solscanner.py


`solscanner.py` is een parallelle SOCKS5 Nmap-scanner geschreven in Python3. Written by Pin3apple (KZ) for Sollie.

Het ondersteunt:

- **Rotatie-proxies** (uit `rotation_proxies.txt`)  
- **Webshare-proxies** (uit `webshare_proxies.txt`)  
- **WireGuard-configuraties**  
- **No-VPN** (via je eigen IP)
  

Deze README richt zich op de twee primaire modi:

- **rotation** (automatische proxyrotatie via `rotation_proxies.txt`)  
- **proxy** (vaste proxy-lijst uit `webshare_proxies.txt`, ideaal voor Webshare.com)

---

## Inhoudsopgave

1. [Vereisten](#vereisten)  
2. [Configuratiebestanden](#configuratiebestanden)  
3. [Eerste installatie](#eerste-installatie)  
4. [Proxy-modi](#proxy-modi)  
   - [Rotatie-modus](#rotatie-modus)  
   - [Proxy-modus](#proxy-modus)  
5. [Overige modi](#overige-modi)  
6. [Opties](#opties)  
7. [Scanresultaten & logs](#scanresultaten--logs)  
8. [Tips & best practices](#tips--best-practices)  

---

## Vereisten

- **Python** 3.x  
- Python-libraries:
  ```bash
  pip install pysocks requests
  

* Systeemtools:

  * `proxychains4`
  * `nmap`
  * `curl`
  * *(optioneel voor WireGuard)* `wireguard-tools`

> **Tip:**
> Run de ingebouwde installer om alles automatisch te installeren:
>
> ```bash
> sudo python3 solscanner.py --init
> ```

---

## Configuratiebestanden

1. **Doelenlijst** (`target.txt`)

   * Één host of IP per regel, bijv.:

     ```
     example.com
     192.168.1.10
     ```

2. **Rotatie-proxies** (`rotation_proxies.txt`)

   * Voor `--mode rotation`
   * Één proxy per regel, formaat:

     ```
     user:pass@host:port
     ```

   * Voorbeeld:

     ```
     alice:pwd123@1.2.3.4:1080
     bob:secure@5.6.7.8:1080
     ```

3. **Webshare-proxies** (`webshare_proxies.txt`)

   * Voor `--mode proxy` (export vanuit Webshare.com API of dashboard)
   * Zelfde formaat als hierboven.

4. **WireGuard-configuraties** (`wireguard/*.conf`)

   * Voor `--mode wireguard` (minimaal twee `.conf`-bestanden nodig om te kunnen roteren).

5. **Logs-map** (`logs/`)

   * Wordt automatisch aangemaakt; individuele scan-logs worden hier opgeslagen.

---

## Eerste installatie

```bash
sudo python3 solscanner.py --init
```

Dit doet:


1. Installeert de systeem-pakketten:

   ```bash
   apt update && apt install -y proxychains4 nmap curl wireguard-tools python3-pip python3-requests
   ```

2. Vraagt je om op **Enter** te drukken om af te sluiten.
3. Geeft aan waar je de proxy-bestanden moet plaatsen (`rotation_proxies.txt`, `webshare_proxies.txt`).

---

## Proxy-modes

### Rotatie-modus

* Leest proxies uit `rotation_proxies.txt`
* Voert een snelle health-check uit (maakt verbinding met Google)
* Kies willekeurig één werkende proxy per thread
* Genereert een tijdelijke `proxychains4`-config (`.proxychains4_<tid>.conf`)
* Voert de scan uit:

  ```bash
  proxychains4 -f .proxychains4_<tid>.conf nmap <nmap-opties> <target>
  ```

* Verifieert de exit-IP via `curl ifconfig.me`

**Voorbeeld:**

```bash
python3 solscanner.py \
  --mode rotation \
  --workers 10 \
  --target-file target.txt \
  --nmap-cmd "-sT -sV -Pn" \
  --preports 80 443
```

### Proxy-modus

* Leest proxies uit `webshare_proxies.txt`
* Voert health-checks uit zoals in rotatie-modus
* Rouleert door de lijst in volgorde; na uitputting start je opnieuw
* Zelfde `proxychains4`-workflow als bij rotatie
* **Let op:** Exit-IP kan niet automatisch geverifieerd worden

**Voorbeeld:**

```bash
python3 solscanner.py \
  --mode proxy \
  --workers 5 \
  --target-file target.txt \
  --nmap-cmd "-sT -sV -Pn" \
  --preports 80 443
```

---

## Overige modes

* **wireguard**: roteert door `wireguard/*.conf`-bestanden
* **no-vpn**: scant rechtstreeks via je eigen IP (zonder proxy)

---

## Opties

| Optie               | Beschrijving                                              |
| ------------------- | --------------------------------------------------------- |
| `--mode`            | `rotation` / `proxy` / `wireguard` / `no-vpn`             |
| `--workers <N>`     | Aantal parallelle threads (standaard = aantal targets)    |
| `--interactive`     | Prompt (y/n) voor bevestiging vóór elke scan              |
| `--stealth`         | Wacht 3–12 sec. tussen scans voor stealth-modus           |
| `--preports P1 P2…` | Poorten voor bereikbaarheidstest (standaard: `443`, `80`) |
| `--nmap-cmd CMD`    | Nmap-opties, bv. `"-sS -Pn -T4"`                          |
| `--target-file F`   | Pad naar doelenlijst (standaard: `target.txt`)            |

---

## Scanresultaten & logs

Na afloop zie je een samenvatting zoals:

```
[✓] example.com | exit 1.2.3.4:1080 (123.123.123.123) | port 443 | log: logs/example.com_1.2.3.4-20250617-143200.log
[x] 192.168.1.10 SKIPPED (Unreachable)
```

* **✓ Scanned**: succesvol
* **x Skipped**: overgeslagen (reden weergegeven)

Individuele logs vind je in de `logs/`-map.

---

## Tips & best practices

* **Houd proxy-lijsten actueel**: de Webshare.com API kan ze automatisch verversen.
* **Voer health-checks uit**: roep periodiek de functie `health_check_proxies()` aan.
* **Switch modi**: als `rotation` niet genoeg exit-IP-diversiteit biedt, probeer `--mode proxy`.
* **Scan alleen met toestemming**: voer scans alleen uit op hosts/netwerken waarvoor je expliciete toestemming hebt.
