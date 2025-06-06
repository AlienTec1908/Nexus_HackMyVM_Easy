# Nexus - HackMyVM (Medium)
 
![Nexus.png](Nexus.png)

## Übersicht

*   **VM:** Nexus
*   **Plattform:** https://hackmyvm.eu/machines/machine.php?vm=Nexus
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 6. Juni 2025
*   **Original-Writeup:** https://alientec1908.github.io/Nexus_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Dieses Writeup dokumentiert den Lösungsweg für die "Nexus"-Maschine auf der HackMyVM-Plattform. Der Weg zum Root-Zugriff umfasste eine detaillierte Web-Enumeration, die zur Entdeckung von versteckten Hinweisen in Form von kodierten Nachrichten im HTML-Quellcode führte. Diese Hinweise offenbarten eine kritische SQL-Injection-Schwachstelle in einem Login-Formular. Durch die Ausnutzung dieser Schwachstelle konnten SSH-Zugangsdaten extrahiert werden. Der finale Schritt zur Privilegienerweiterung erfolgte durch den Missbrauch einer unsicheren `sudo`-Konfiguration für den `find`-Befehl.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `nikto`
*   `gobuster`
*   `curl`
*   `sqlmap`
*   Standard Linux-Befehle (`vi`, `ssh`, `sudo`, `find`, `strings`, `ls`, `cat`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Nexus" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration:**
    *   Die IP-Adresse des Ziels wurde schnell mit `arp-scan` identifiziert. Ein umfassender `nmap`-Scan enthüllte zwei offene Ports: Port 22 (SSH - OpenSSH 9.2p1) und Port 80 (HTTP - Apache 2.4.62).

2.  **Web Enumeration:**
    *   `nikto` und `gobuster` wurden eingesetzt, um die Webanwendung zu untersuchen. Dabei wurden mehrere Seiten gefunden, darunter `login.php` und eine inhaltlich auffällige `index2.php`.
    *   Die Analyse des Quellcodes von `index2.php` förderte einen Hex-kodierten String und weitere textliche Hinweise zutage. Die Dekodierung der Nachricht enthüllte einen Hinweis auf einen `eval`-Mechanismus und das Schlüsselwort "pandora".

3.  **Initial Access (SQL-Injection):**
    *   Nachdem eine versteckte `auth-login.php` gefunden, aber als Sackgasse identifiziert wurde, konzentrierte sich der Angriff auf `login.php`.
    *   Ein einfaches Anführungszeichen (`'`) im Benutzerfeld verursachte einen SQL-Fehler, der eine klassische SQL-Injection-Schwachstelle bestätigte.
    *   Mit `sqlmap` wurde die Datenbank `sion` ausgelesen. Aus der Tabelle `users` konnten die SSH-Zugangsdaten für den Benutzer `shelly` (`shelly:cambiame08`) extrahiert werden.
    *   Der Login via SSH mit diesen Credentials war erfolgreich.

4.  **Post-Exploitation / Privilege Escalation (von shelly zu root):**
    *   Nach dem Login wurde mit `sudo -l` die `sudo`-Konfiguration des Benutzers `shelly` überprüft.
    *   Es wurde festgestellt, dass `shelly` den Befehl `/usr/bin/find` als Root ohne Passworteingabe (`NOPASSWD`) ausführen darf.

5.  **Privilege Escalation (von shelly zu root):**
    *   Die unsichere `sudo`-Regel wurde ausgenutzt, indem mit `sudo find . -exec /bin/sh \; -quit` eine Root-Shell gestartet wurde. Damit wurden volle administrative Rechte auf dem System erlangt.

## Wichtige Schwachstellen und Konzepte

*   **Information Leakage in HTML Source:** Im Quellcode der Webseite waren kritische Hinweise in Form von kodierten Nachrichten und Klartext-Phrasen versteckt, die direkt zur nächsten Schwachstelle führten.
*   **SQL-Injection (Error-Based & Blind):** Eine unsachgemäß validierte Benutzereingabe im Login-Formular erlaubte die Injektion von SQL-Befehlen, was zur Umgehung der Authentifizierung und zur Extraktion von sensiblen Daten (Benutzernamen und Passwörter) führte.
*   **Unsichere Sudo-Konfiguration:** Die Erlaubnis, einen mächtigen Befehl wie `find` mit `sudo`-Rechten und `NOPASSWD` auszuführen, ist eine klassische Fehlkonfiguration, die eine einfache und direkte Eskalation zum Root-Benutzer ermöglicht (siehe GTFOBins).
*   **Password Re-use:** Das in der Web-Datenbank gefundene Passwort wurde für den System-Login (SSH) wiederverwendet, was den initialen Zugriff erst ermöglichte.

## Flags

*   **User Flag (`SA/user-flag.txt`):** `82.................gd`
*   **Root Flag (extrahiert aus `Sion-Code/use-fim-to-root.png`):** `HMV-FLAG[[ p3....2s9 ]]`

## Tags

`HackMyVM`, `Nexus`, `Medium`, `SQL-Injection`, `Sudo-Exploitation`, `Linux`, `Web`, `Privilege Escalation`
