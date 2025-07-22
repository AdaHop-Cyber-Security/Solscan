2025 All rights reserved - written by Kevin Zwaan @Pin3apple

---

# SolWalk - A Advanced Network Attack Orchestrator

[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Status](https://img.shields.io/badge/status-active-success.svg)]()

A tool for cyber security professionals by cyber security professionals - designed for Sollie.

---

## Inhoudsopgave

- [Belangrijkste Kenmerken](#belangrijkste-kenmerken)
- [Hoe het Werkt](#hoe-het-werkt)
- [Vereisten](#vereisten)
- [Installatie & Configuratie](#installatie--configuratie)
  - [Stap 1: Installatie](#stap-1-installatie)
  - [Stap 2: Configuratie (`config.ini`)](#stap-2-configuratie-configini)
  - [Stap 3: Doelen & Proxy's](#stap-3-doelen--proxys)
- [Gebruik (Usage)](#gebruik-usage)
  - [Argumenten](#argumenten)
- [Voorbeelden](#voorbeelden)
- [De Output Begrijpen](#de-output-begrijpen)
- [Disclaimer](#disclaimer)

## Belangrijkste Kenmerken

- **Multi-Module Framework**: Voer diverse scans uit, van standaard Nmap-verkenning tot geavanceerde firewall-ontwijkingstechnieken en live kwetsbaarheidsscans.
  - **Nieuw - `vuln-scan` Module**: Voert een diepe Nmap-scan uit en controleert de gevonden softwareversies direct tegen de live **Vulners API** voor bekende CVE's.
- **Geavanceerd Anonymity Framework**: Routeer verkeer via:
  - **WireGuard**: Automatisch verbinden en rouleren van WireGuard VPN-tunnels op een thread-veilige manier.
  - **SOCKS5 Proxies**: Gebruik statische of roterende SOCKS5-proxy's met `proxychains4`.
  - **Direct**: Standaard directe verbindingen (met interactieve waarschuwing).
- **Flexibele Configuratie**: Beheer alle paden en API-sleutels via een eenvoudig `config.ini` bestand. Geen aanpassingen in de code nodig.
- **Geavanceerde Debugging**: Gebruik `-v` of `-vv` om precies te zien wat het script doet, van de Nmap-commando's tot de ruwe XML-output en API-payloads.
- **Parallelle Uitvoering**: Gebruikt een thread pool om aanvallen op meerdere doelen tegelijk uit te voeren, wat de efficiëntie drastisch verhoogt.
- **Geautomatiseerde Installatie**: Een `--init` vlag om alle systeem- en Python-afhankelijkheden automatisch te installeren.
- **Gedetailleerde Logging**: Elke aanval wordt opgeslagen in een uniek logbestand met details over de exit-node, tijd en resultaten.

## Hoe het Werkt

SolWalk beheert een pool van "exit nodes" (WireGuard-configuraties of SOCKS5-proxy's) en een lijst van doelen. Voor elk doel wijst een worker-thread een exit-node toe, stelt de netwerkomgeving in (bijv. het opzetten van een VPN-tunnel) en voert de geselecteerde scan uit. Na de taak wordt de omgeving weer opgeruimd. De `vuln-scan` module parseert de Nmap-output, extraheert software- en OS-informatie, en stuurt deze naar de Vulners API voor een real-time kwetsbaarheidsanalyse.

## Vereisten

- **Besturingssysteem**: Een op Debian gebaseerd Linux-systeem (bijv. Kali Linux, Ubuntu).
- **Rechten**: `sudo` of `root`-toegang is vereist voor het installeren van pakketten, het beheren van WireGuard-interfaces en het uitvoeren van bepaalde Nmap-scans.
- **API-sleutel (optioneel)**: Voor de `vuln-scan` module is een gratis API-sleutel van [vulners.com](https://vulners.com/) vereist.

## Installatie & Configuratie

### Stap 1: Installatie

1.  **Kloon de repository:**
    ```bash
    git clone https://github.com/jouw-gebruiker/jouw-repo.git
    cd jouw-repo
    ```

2.  **Voer de automatische installatie uit:**
    Deze stap installeert `nmap`, `proxychains4`, `wireguard-tools` en alle benodigde Python-bibliotheken.
    ```bash
    sudo python3 solwalk.py --init
    ```

### Stap 2: Configuratie (`config.ini`)

Maak een bestand genaamd `config.ini` in dezelfde map als het script. Dit bestand centraliseert al je instellingen.

```ini
[paths]
# Locatie voor WireGuard .conf bestanden
wg_config_dir = /home/kali/sollie/wireguard

# Standaard bestand met doelwitten
target_file = target.txt

# Map waar logs worden opgeslagen
log_dir = logs

# Bestand met roterende SOCKS5 proxies
proxy_list_rotation = rotation_proxies.txt

# Bestand met statische SOCKS5 proxies
proxy_list_static = webshare_proxies.txt

[api]
# API-sleutel voor de vuln-scan module
vulners_api_key = JOUW_VULNERS_API_SLEUTEL_HIER
```
*Tip: Je kunt de API-sleutel ook als omgevingsvariabele instellen: `export VULNERS_API_KEY='jouw_sleutel'`*

### Stap 3: Doelen & Proxy's

Maak de bestanden aan zoals gespecificeerd in je `config.ini`.

1.  **Doelen (`target.txt`)**: Eén doel (IP of domein) per regel.
2.  **Proxy-lijsten**: `rotation_proxies.txt` en/of `webshare_proxies.txt` met proxy's in `user:pass@host:port` formaat.
3.  **WireGuard-map**: Plaats je `.conf` bestanden in de map die je hebt ingesteld in `wg_config_dir`.

## Gebruik (Usage)

De tool wordt volledig via de command-line bediend.

```bash
sudo python3 solwalk.py [TARGETS] [ATTACK] [MODE] [OPTIONS]
```

### Argumenten

| Groep                        | Argument             | Beschrijving                                                              |
| ---------------------------- | -------------------- | ------------------------------------------------------------------------- |
| **General Options**          | `-h`, `--help`         | Toon dit helpbericht en sluit af.                                         |
|                              | `--init`             | Installeer alle afhankelijkheden en sluit af.                             |
|                              | `-v`, `-vv`          | Verhoog het detailniveau van de output (info, debug).                     |
| **Target Specification**     | `-t`, `--target`       | Een enkel doel-IP-adres of hostnaam.                                      |
|                              | `-tf`, `--target-file` | Een bestand met doelen (standaard: `target.txt`).                         |
| **Attack Configuration**     | `--attack-type`      | Het type aanval (`nmap`, `vuln-scan`, `scapy-*`).                           |
|                              | `--nmap-cmd`         | De Nmap-commando-string die moet worden uitgevoerd.                       |
|                              | `--attack-port`      | De doelpoort voor Scapy- en Socket-aanvallen (standaard: 443).             |
| **Execution & Anonymity**    | `--workers`          | Het aantal parallelle aanvalsthreads (standaard: aantal doelen).           |
|                              | `--mode`             | De routeringsmethode (`direct`, `wireguard`, `proxy`, `rotation`).          |
|                              | `--timeout`          | Timeout in seconden voor Nmap-scans (standaard: 600).                     |
|                              | `--run-when-up`      | Controleer proxy's "on-the-fly" in plaats van vooraf.                   |
|                              | `--overwrite`        | Forceert de scan, zelfs als er een VPN-lek wordt gedetecteerd.            |

## Voorbeelden

**1. Live Kwetsbaarheidsscan (Nieuw!)**
Scan een doelwit, detecteer services en OS (`-sV -O`), en controleer deze direct op bekende CVE's via de Vulners API.
```bash
sudo python3 solwalk.py -t 185.145.27.105 --attack-type vuln-scan --nmap-cmd "-sT -sV -O -p 80,443 -T3"
```

**2. Meerdere doelen scannen via WireGuard**
Scan alle doelen uit `target.txt` met 5 parallelle workers, waarbij elke worker een unieke WireGuard-tunnel gebruikt.
```bash
sudo python3 solwalk.py -tf target.txt --mode wireguard --workers 5
```

**3. Debugging Sessie**
Voer een `vuln-scan` uit met maximale output om precies te zien welke commando's worden uitgevoerd en welke data naar de API wordt gestuurd.
```bash
sudo python3 solwalk.py -t scanme.nmap.org --attack-type vuln-scan -vv```

**4. Nmap-scan via Roterende SOCKS5-proxy's**
Voer een Nmap TCP-scan (`-sT`) uit op de top 20 poorten, gebruikmakend van 10 workers die roteren door de proxy's in `rotation_proxies.txt`.
```bash
sudo python3 solwalk.py -tf target.txt --mode rotation --workers 10 --nmap-cmd "-sT -Pn --top-ports 20"
```

## De Output Begrijpen

De samenvatting aan het einde geeft de status van elke taak weer:

```
==================== ATTACK RUN SUMMARY ====================
[✓] 185.145.27.105   | SUCCESS      | Exit: (SOCKS5) via 166.0.68.90:6550  | Details: Vulnerabilities found. Full report in log.
[✓] 192.168.1.10     | Completed    | Exit: wg-nl-01.conf                  | Details: Nmap scan logged to logs/192.168.1.10_nmap_...
[x] 10.0.0.5         | Failed       | Exit: No Proxy                       | Details: No working proxy available.
================================================================
```

- **`[✓]` (Groen)**: De taak is voltooid.
  - `SUCCESS`: De `vuln-scan` heeft kwetsbaarheden gevonden.
  - `Completed`: De taak is succesvol afgerond (bv. Nmap-scan voltooid, geen CVE's gevonden).
- **`[x]` (Rood)**: De taak is mislukt (`Failed`) of overgeslagen (`Skipped`). De details geven de reden aan.

## Disclaimer

**Deze tool is uitsluitend bedoeld voor educatieve doeleinden en geautoriseerde security-audits.** Het gebruik van deze tool tegen netwerken of systemen zonder expliciete voorafgaande toestemming is illegaal. De ontwikkelaar aanvaardt geen aansprakelijkheid en is niet verantwoordelijk voor misbruik of schade veroorzaakt door dit programma.

### Belangrijke operationele opmerkingen:

- **Scapy & Proxies**: Scapy-aanvallen (`scapy-overlap`, etc.) werken op een laag niveau (Layer 3) en **zullen SOCKS5-proxy's omzeilen**. Ze worden altijd verzonden via de actieve netwerkinterface (bijv. `eth0` of een WireGuard-interface).
- **VPN-lekken**: De `--overwrite` vlag is gevaarlijk en kan uw echte IP-adres blootstellen. Gebruik deze alleen als u de risico's volledig begrijpt.
- **Als je de melding ```'subprocess.CalledProcessError: Command '['/usr/bin/python3', '-m', 'pip', 'install', '--upgrade', 'pip']' returned non-zero exit status 1.``` '** krijgt kun je via venv pip upgraden. Gebruik pip nooit buiten een virtual enviroment!

  ```bash
  source venv/bin/activate
  python -m ensurepip --upgrade
  python -m pip install --upgrade pip
  ```

  ```bash
  pip install requests
  ```
