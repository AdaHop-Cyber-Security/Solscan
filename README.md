2025 All rights reserved - written by Kevin Zwaan @Pin3apple

---

# SolWalk - Advanced Nmap evasive parallel scanner

[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Status](https://img.shields.io/badge/status-active-success.svg)]()

A tool for cyber security professionals by cyber security professionals - designed for Sollie 

---

## Inhoudsopgave

- [Belangrijkste Kenmerken](#belangrijkste-kenmerken)
- [Hoe het Werkt](#hoe-het-werkt)
- [Vereisten](#vereisten)
- [Installatie](#installatie)
- [Configuratie](#configuratie)
- [Gebruik](#gebruik-Usage)
  - [Argumenten](#argumenten)
- [Voorbeelden](#voorbeelden)
  - [Voorbeeld 1: Standaard Nmap-scan](#voorbeeld-1-standaard-nmap-scan)
  - [Voorbeeld 2: Meerdere doelen scannen via WireGuard](#voorbeeld-2-meerdere-doelen-scannen-via-wireguard)
  - [Voorbeeld 3: Geavanceerde firewall-ontwijkingstest](#voorbeeld-3-geavanceerde-firewall-ontwijkingstest)
  - [Voorbeeld 4: Nmap-scan via roterende SOCKS5-proxy's](#voorbeeld-4-nmap-scan-via-roterende-socks5-proxys)
- [De Output Begrijpen](#de-output-begrijpen)
- [Disclaimer](#disclaimer)

## Belangrijkste Kenmerken

- **Multi-Attack Modules**: Voer verschillende soorten aanvallen uit, van standaard Nmap-scans tot geavanceerde Layer 3/4-ontwijkingstechnieken.
- **Geavanceerd Anonymity Framework**: Routeer verkeer via:
  - **WireGuard**: Automatisch verbinden en rouleren van WireGuard VPN-tunnels.
  - **SOCKS5 Proxies**: Gebruik statische of roterende SOCKS5-proxy's met `proxychains4`.
  - **Direct**: Standaard directe verbindingen.
- **VPN Lekbeveiliging**: Een ingebouwde veiligheidscontrole stopt de uitvoering als het IP-adres van de VPN lekt, tenzij handmatig overschreven.
- **Parallelle Uitvoering**: Gebruikt een thread pool om aanvallen op meerdere doelen tegelijk uit te voeren, wat de efficiëntie drastisch verhoogt.
- **Geautomatiseerde Installatie**: Een `--init` vlag om alle systeem- en Python-afhankelijkheden automatisch te installeren.
- **Gedetailleerde Logging**: Elke aanval wordt opgeslagen in een uniek logbestand met details over de exit-node, tijd en resultaten.

## Hoe het Werkt

De orchestrator beheert een pool van "exit nodes" (WireGuard-configuraties of SOCKS5-proxy's) en een lijst van doelen. Voor elk doel wijst een worker-thread een exit-node toe, stelt de netwerkomgeving in (bijv. het opzetten van een VPN-tunnel) en voert de geselecteerde aanval uit. Na de aanval wordt de omgeving weer opgeruimd, bijv. de VPN-verbinding wordt verbroken. Dit zorgt ervoor dat elke aanval geïsoleerd en via een andere route kan worden uitgevoerd.

## Vereisten

- **Besturingssysteem**: Een op Debian gebaseerd Linux-systeem (bijv. Kali Linux, Ubuntu).
- **Rechten**: `sudo` of `root`-toegang is vereist voor het installeren van pakketten, het beheren van WireGuard-interfaces en het uitvoeren van Scapy-aanvallen.

## Installatie

1.  **Kloon de repository:**
    ```bash
    git clone https://github.com/jouw-gebruiker/jouw-repo.git
    cd jouw-repo
    ```

2.  **Voer de automatische installatie uit:**
    Deze stap installeert `nmap`, `proxychains4`, `wireguard-tools` en alle benodigde Python-bibliotheken.
    ```bash
    sudo python3 orchestrator.py --init
    ```
    Volg daarna de instructies om uw configuratiebestanden aan te maken.

## Configuratie

Voordat u de tool kunt gebruiken, moet u de volgende mappen en bestanden aanmaken in dezelfde directory als het script:

1.  **Doelen (`target.txt`)**:
    Maak een bestand genaamd `target.txt` en voeg één doel, IP-adres of domein, per regel toe.
    ```
    192.168.1.1
    scanme.nmap.org
    10.0.0.5
    ```

2.  **WireGuard-configuraties (optioneel)**:
    Maak een map genaamd `wireguard` en plaats al uw `.conf` bestanden voor WireGuard hierin.
    ```
    /pad/naar/script/
    ├── wireguard/
    │   ├── wg-nl-01.conf
    │   ├── wg-us-02.conf
    │   └── wg-de-03.conf
    └── orchestrator.py
    ```
    *Opmerking: het script gebruikt het pad `/home/kali/sollie/wireguard`. Pas de `WG_CONFIG_DIR` variabele in het script aan als uw pad anders is.*

3.  **Proxy-lijsten (optioneel)**:
    Maak `rotation_proxies.txt` en/of `webshare_proxies.txt`. De proxy's moeten in het `user:pass@host:port` formaat zijn.
    *`rotation_proxies.txt`*:
    ```
    gebruiker1:wachtwoord123@proxy.example.com:8080
    gebruiker2:wachtwoord456@123.45.67.89:9090
    ```

## Gebruik

De tool wordt volledig via de command-line bediend.

```bash
sudo python3 orchestrator.py --mode [MODE] --attack-type [ATTACK] [TARGET_OPTIONS]
```

### Argumenten

| Groep                        | Argument             | Beschrijving                                                                                                              | Standaardwaarde                |
| ---------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| **Required Arguments**       | `-t`, `--target`       | Een enkel doel-IP-adres.                                                                                                  | -                              |
|                              | `-tf`, `--target-file` | Een bestand met doelen, één per regel.                                                                                    | `target.txt`                   |
| **Attack Configuration**     | `--attack-type`      | Het type aanval: `nmap`, `scapy-overlap`, `scapy-tinyfrag`, `scapy-invalidflags`, `http-smuggle`.                             | `nmap`                         |
|                              | `--nmap-cmd`         | De Nmap-commando-string die moet worden uitgevoerd.                                                                       | `-sS -Pn -T4 --top-ports 1000` |
|                              | `--attack-port`      | De doelpoort voor Scapy- en Socket-aanvallen.                                                                             | `443`                          |
| **Execution & Anonymity**    | `--workers`          | Het aantal parallelle aanvalsthreads.                                                                                     | `1`                            |
|                              | `--mode`             | De routeringsmethode: `direct`, `wireguard`, `proxy`, `rotation`.                                                           | `direct`                       |
|                              | `--stealth`          | Voegt een willekeurige vertraging tussen taken toe.                                                                       | Uitgeschakeld                  |
|                              | `--overwrite`        | Forceert de scan, zelfs als er een VPN-lek wordt gedetecteerd. **GEBRUIK MET VOORZICHTIGHEID!**                             | Uitgeschakeld                  |
| **Setup**                    | `--init`             | Installeert alle afhankelijkheden en sluit af.                                                                            | -                              |

## Voorbeelden

### Voorbeeld 1: Standaard Nmap-scan

Voer een eenvoudige Nmap SYN-scan uit tegen een enkel doel via een directe verbinding.
```bash
sudo python3 orchestrator.py --attack-type nmap -t scanme.nmap.org
```

### Voorbeeld 2: Meerdere doelen scannen via WireGuard

Scan alle doelen uit `target.txt` met 5 parallelle workers, waarbij elke worker een willekeurige WireGuard-configuratie uit de `wireguard/` map gebruikt.
```bash
sudo python3 orchestrator.py --mode wireguard --workers 5 -tf target.txt
```

### Voorbeeld 3: Geavanceerde firewall-ontwijkingstest

Test een webserver op poort 80 tegen een "tiny fragment" aanval. Deze aanval vereist `sudo`.
```bash
sudo python3 orchestrator.py --attack-type scapy-tinyfrag --attack-port 80 -t 192.168.1.10
```

### Voorbeeld 4: Nmap-scan via roterende SOCKS5-proxy's

Voer een Nmap TCP-scan (`-sT`) uit op de top 20 poorten tegen meerdere doelen, gebruikmakend van 10 workers die roteren door de proxy's in `rotation_proxies.txt`.
```bash
sudo python3 orchestrator.py --mode rotation --workers 10 -tf target.txt --nmap-cmd "-sT -Pn --top-ports 20"
```

## De Output Begrijpen

Aan het einde van de uitvoering wordt een samenvatting getoond:

```
==================== ATTACK RUN SUMMARY ====================
[✓] scanme.nmap.org  | Completed    | Exit: direct                       | Details: Nmap scan logged to logs/scanme.nmap.org_nmap_20231027-143000.log
[✓] 192.168.1.10     | SUCCESS      | Exit: wg-nl-01.conf                | Details: CRITICAL: 'tiny fragment' attack successful. Port is OPEN.
[x] 10.0.0.5         | Failed       | Details: VPN leak detected
================================================================
```

- **`[✓]`**: Geeft aan dat de taak succesvol is uitgevoerd. De status kan zijn:
  - `Completed`: De taak (bv. Nmap-scan) is voltooid. Details staan in het logbestand.
  - `SUCCESS`: De specifieke aanval (bv. Scapy) was succesvol en heeft een kwetsbaarheid blootgelegd.
  - `Response`: De aanval ontving een onverwachte maar interessante reactie.
- **`[x]`**: Geeft aan dat de taak is mislukt of overgeslagen (`Failed`, `Skipped`). De details geven de reden aan.

## Disclaimer

**Deze tool is uitsluitend bedoeld voor educatieve doeleinden en geautoriseerde security-audits.**

Het gebruik van deze tool tegen netwerken of systemen zonder expliciete voorafgaande toestemming is illegaal. De ontwikkelaar aanvaard geen aansprakelijkheid en is niet verantwoordelijk voor misbruik of schade veroorzaakt door dit programma.

**Belangrijke operationele opmerkingen:**

- **Scapy & Proxies**: Scapy-aanvallen (`scapy-overlap`, etc.) werken op een laag niveau (Layer 3) en **zullen SOCKS5-proxy's omzeilen**. Ze worden altijd verzonden via de actieve netwerkinterface (bijv. `eth0` of een WireGuard-interface).
- **VPN-lekken**: De `--overwrite` vlag is gevaarlijk en kan uw echte IP-adres blootstellen. Gebruik deze alleen als u de risico's volledig begrijpt.
