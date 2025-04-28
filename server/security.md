# Secure Linux Server Cheat Sheet

## 1. Root-Login deaktivieren

*   **Warum:** Verhindert direkte Brute-Force-Angriffe auf den Root-Account (höchste Rechte).
*   **Wie:**
    1.  SSH-Konfigurationsdatei öffnen:
        ```bash
        sudo nano /etc/ssh/sshd_config
        ```
    2.  Zeile `PermitRootLogin yes` ändern zu:
        ```
        PermitRootLogin no
        ```
    3.  SSH-Dienst neu starten:
        ```bash
        sudo systemctl restart sshd
        ```

---

## 2. Schlüsselbasierte SSH-Authentifizierung verwenden

*   **Warum:** SSH-Schlüssel sind wesentlich sicherer und schwerer zu knacken als Passwörter.
*   **Wie:**
    1.  Schlüsselpaar auf dem lokalen Rechner erstellen:
        ```bash
        ssh-keygen -t rsa -b 4096
        ```
    2.  Öffentlichen Schlüssel auf den Server kopieren:
        ```bash
        ssh-copy-id username@server_ip
        ```
    3.  Passwort-Authentifizierung auf dem Server deaktivieren (`/etc/ssh/sshd_config`):
        ```
        PasswordAuthentication no
        ```
    4.  SSH-Dienst neu starten:
        ```bash
        sudo systemctl restart sshd
        ```

---

## 3. Starke Passwortrichtlinien erzwingen

*   **Warum:** Verhindert schwache, leicht zu erratende Passwörter und reduziert das Risiko von Brute-Force-Angriffen.
*   **Wie:**
    1.  Passwortrichtlinien-Datei bearbeiten:
        ```bash
        sudo nano /etc/security/pwquality.conf
        ```
    2.  Richtlinien festlegen (Beispiel):
        ```
        minlen = 12
        minclass = 3
        ```
        *   `minlen`: Mindestlänge (z.B. 12 Zeichen).
        *   `minclass`: Erfordert verschiedene Zeichentypen (Groß-/Kleinbuchstaben, Ziffern, Sonderzeichen).

---

## 4. System aktuell halten

*   **Warum:** Updates schließen bekannte Sicherheitslücken.
*   **Wie:**
    *   System aktualisieren:
        ```bash
        # Debian/Ubuntu
        sudo apt update && sudo apt upgrade -y

        # CentOS/RHEL
        sudo yum update -y
        ```
    *   (Optional) Automatische Updates aktivieren (Ubuntu):
        ```bash
        sudo apt install unattended-upgrades
        ```

---

## 5. Firewall konfigurieren (ufw Beispiel)

*   **Warum:** Limitiert den Zugriff auf notwendige Dienste und blockiert unautorisierten Traffic.
*   **Wie (Ubuntu mit ufw):**
    1.  Ufw installieren (falls nicht vorhanden):
        ```bash
        sudo apt install ufw
        ```
    2.  Benötigte Ports freigeben (Beispiele):
        ```bash
        sudo ufw allow 22    # SSH
        sudo ufw allow 80    # HTTP
        sudo ufw allow 443   # HTTPS
        ```
    3.  Firewall aktivieren:
        ```bash
        sudo ufw enable
        ```

---

## 6. Intrusion Detection installieren (Fail2Ban)

*   **Warum:** Schützt vor Brute-Force-Angriffen, indem IPs nach zu vielen Fehlversuchen blockiert werden.
*   **Wie:**
    1.  Fail2Ban installieren:
        ```bash
        sudo apt install fail2ban # Debian/Ubuntu
        # sudo yum install fail2ban # CentOS/RHEL
        ```
    2.  Konfiguration anpassen (typischerweise in `/etc/fail2ban/jail.local` statt `jail.conf`):
        ```bash
        sudo nano /etc/fail2ban/jail.local
        ```
    3.  SSH-Überwachung aktivieren/anpassen (Beispiel in `jail.local`):
        ```ini
        [sshd]
        enabled = true
        maxretry = 5
        bantime = 3600 ; # 1 Stunde
        ```
    4.  Fail2Ban neu starten:
        ```bash
        sudo systemctl restart fail2ban
        ```

---

## 7. Unnötige Dienste deaktivieren

*   **Warum:** Reduziert die Angriffsfläche, indem weniger potenzielle Einstiegspunkte aktiv sind.
*   **Wie:**
    1.  Aktive Dienste auflisten:
        ```bash
        sudo systemctl list-unit-files --type=service --state=enabled
        ```
    2.  Nicht benötigte Dienste deaktivieren:
        ```bash
        sudo systemctl disable <service_name>
        sudo systemctl stop <service_name>
        ```

---

## 8. Korrekte Dateiberechtigungen setzen

*   **Warum:** Schützt sensible Dateien vor unautorisiertem Zugriff oder Änderung.
*   **Wie (Beispiele):**
    ```bash
    sudo chmod 600 /etc/ssh/sshd_config
    sudo chmod 640 /var/log/auth.log # Nur root und adm Gruppe können lesen
    ```

---

## 9. Logging und Monitoring aktivieren

*   **Warum:** Protokolle helfen, ungewöhnliche Aktivitäten zu erkennen und Vorfälle zu analysieren.
*   **Wie:**
    *   Standardmäßig ist `rsyslog` meist aktiv. Konfiguration in `/etc/rsyslog.conf` und `/etc/rsyslog.d/`.
    *   Erwäge zentrale Logging-Lösungen (z.B. ELK Stack, Graylog).

---

## 10. Auditing mit `auditd` implementieren

*   **Warum:** Überwacht kritische Dateien und Aktionen, alarmiert bei Änderungen.
*   **Wie:**
    1.  `auditd` installieren:
        ```bash
        sudo apt install auditd # Debian/Ubuntu
        # sudo yum install audit # CentOS/RHEL
        ```
    2.  Regeln in `/etc/audit/rules.d/audit.rules` hinzufügen (Beispiel: Passwortdatei überwachen):
        ```
        -w /etc/passwd -p wa -k passwd_changes
        ```
    3.  `auditd` neu starten:
        ```bash
        sudo systemctl restart auditd
        ```

---

## 11. SSH-Konfiguration härten

*   **Warum:** Zusätzliche SSH-Einstellungen zur Abwehr von Angriffen.
*   **Wie (`/etc/ssh/sshd_config`):**
    *   Ändere den Standard-Port (Beispiel):
        ```
        Port 2222
        ```
    *   Nur Protokoll 2 verwenden:
        ```
        Protocol 2
        ```
    *   Passwort-Login deaktivieren (siehe Punkt 2):
        ```
        PasswordAuthentication no
        ```
    *   Weitere Optionen prüfen (z.B. `LoginGraceTime`, `MaxAuthTries`, `AllowUsers`).
    *   Nach Änderungen SSH neu starten:
        ```bash
        sudo systemctl restart sshd
        ```

---

## 12. Kernel-Parameter härten

*   **Warum:** Sichert Netzwerkeinstellungen und entschärft bestimmte Angriffsarten.
*   **Wie (`/etc/sysctl.conf` oder in `/etc/sysctl.d/`):**
    ```
    # Schutz gegen SYN-Flood-Angriffe
    net.ipv4.tcp_syncookies = 1

    # Schutz gegen IP-Spoofing
    net.ipv4.conf.all.rp_filter = 1
    net.ipv4.conf.default.rp_filter = 1

    # Ignoriere Source-Routed Pakete
    net.ipv4.conf.all.accept_source_route = 0
    net.ipv4.conf.default.accept_source_route = 0

    # Ignoriere ICMP Redirects
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv4.conf.default.accept_redirects = 0
    net.ipv4.conf.all.secure_redirects = 0
    net.ipv4.conf.default.secure_redirects = 0

    # Logge verdächtige Pakete
    net.ipv4.conf.all.log_martians = 1
    ```
    *   Änderungen anwenden:
        ```bash
        sudo sysctl -p
        ```

---

## 13. Regelmäßige Backups planen

*   **Warum:** Stellt Datenwiederherstellung nach Angriffen, Fehlern oder Ausfällen sicher.
*   **Wie (Beispiel mit `rsync`):**
    ```bash
    rsync -av --delete /wichtige_daten/ benutzer@backup_server:/backup_pfad/
    ```
    *   Tools wie `tar`, `borgbackup` oder Cloud-Backup-Lösungen verwenden.
    *   Backups automatisieren (z.B. per `cron`).

---

## 14. Ressourcenlimits setzen

*   **Warum:** Verhindert Denial-of-Service (DoS)-Angriffe durch Begrenzung des Ressourcenverbrauchs pro Benutzer.
*   **Wie (`/etc/security/limits.conf`):**
    ```
    # Beispiel: Maximale Anzahl Prozesse für alle Benutzer begrenzen
    * soft nproc 4096
    * hard nproc 8192
    ```

---

## 15. Sicherheits-Scan-Tools verwenden

*   **Warum:** Identifiziert Fehlkonfigurationen und Schwachstellen.
*   **Wie (Beispiel mit Lynis):**
    1.  Lynis installieren:
        ```bash
        sudo apt install lynis # Debian/Ubuntu
        # sudo yum install lynis # CentOS/RHEL
        ```
    2.  System-Scan durchführen:
        ```bash
        sudo lynis audit system
        ```
    *   Andere Tools: OpenVAS, Nessus.

---

## 16. Vor Malware schützen

*   **Warum:** Auch Linux kann Ziel von Malware sein, besonders bei Internetzugang oder Dateifreigaben.
*   **Wie (Beispiel mit ClamAV):**
    1.  ClamAV installieren:
        ```bash
        sudo apt install clamav clamav-daemon # Debian/Ubuntu
        # sudo yum install clamav # CentOS/RHEL
        ```
    2.  Virendefinitionen aktualisieren:
        ```bash
        sudo freshclam
        ```
    3.  Verzeichnis scannen:
        ```bash
        sudo clamscan -r /pfad/zum/verzeichnis
        ```

---

## 17. Multi-Faktor-Authentifizierung (MFA/2FA) aktivieren

*   **Warum:** Fügt eine zweite Verifizierungsebene hinzu und erschwert unautorisierten Zugriff erheblich.
*   **Wie (Beispiel mit Google Authenticator für SSH):**
    1.  PAM-Modul installieren:
        ```bash
        sudo apt install libpam-google-authenticator # Debian/Ubuntu
        ```
    2.  Für den Benutzer einrichten:
        ```bash
        google-authenticator
        ```
    3.  MFA in PAM-Konfiguration für SSH aktivieren (`/etc/pam.d/sshd`):
        *   Zeile hinzufügen (normalerweise am Anfang):
            ```
            auth required pam_google_authenticator.so nullok
            ```
            (`nullok` erlaubt temporär Login ohne Token, bis es eingerichtet ist)
    4.  SSH-Konfiguration anpassen (`/etc/ssh/sshd_config`):
        ```
        ChallengeResponseAuthentication yes
        AuthenticationMethods publickey,password publickey,keyboard-interactive # Beispiel, falls Keys + MFA
        ```
    5.  SSH-Dienst neu starten:
        ```bash
        sudo systemctl restart sshd
        ```

---

## 18. Netzwerksegmentierung implementieren

*   **Warum:** Begrenzt den Datenverkehr zwischen verschiedenen Teilen der Infrastruktur und reduziert die Auswirkungen eines Angriffs.
*   **Wie:**
    *   Cloud: VPCs, Subnetze, Sicherheitsgruppen (AWS, Azure, GCP).
    *   Lokal: VLANs, Firewall-Regeln (`iptables`, `nftables`, `firewalld`).
    *   Beispiel (`iptables`): Nur bestimmter IP erlauben, auf SSH zuzugreifen:
        ```bash
        sudo iptables -A INPUT -p tcp -s TRUSTED_IP --dport 22 -j ACCEPT
        sudo iptables -A INPUT -p tcp --dport 22 -j DROP # Rest blockieren (Vorsicht!)
        ```

---

## 19. `sudo`-Zugriff einschränken

*   **Warum:** Minimiert das Risiko der Rechteausweitung (Privilege Escalation).
*   **Wie:**
    1.  `sudoers`-Datei sicher bearbeiten:
        ```bash
        sudo visudo
        ```
    2.  Spezifische Berechtigungen definieren statt Vollzugriff:
        ```
        # Benutzer 'user1' darf nur apt update/upgrade ausführen
        user1 ALL = /usr/bin/apt update, /usr/bin/apt upgrade

        # Gruppe 'admins' hat vollen Zugriff
        %admins ALL=(ALL) ALL
        ```
    *   Regelmäßig `sudoers` prüfen.

---

## 20. Mandatory Access Control (MAC) erzwingen (AppArmor/SELinux)

*   **Warum:** Beschränkt Prozesse auf benötigte Ressourcen und Aktionen, begrenzt den Schaden bei Kompromittierung.
*   **Wie:**
    *   **AppArmor (Ubuntu/Debian):**
        *   Status prüfen: `sudo aa-status` oder `sudo apparmor_status`
        *   Profile in `/etc/apparmor.d/` anpassen.
        *   Modus ändern (enforce/complain): `sudo aa-enforce /pfad/zum/profil` oder `sudo aa-complain /pfad/zum/profil`
    *   **SELinux (CentOS/RHEL):**
        *   Status prüfen: `sestatus`
        *   Modus ändern (enforcing/permissive/disabled): `sudo setenforce 1` (Enforcing) oder `sudo setenforce 0` (Permissive). Dauerhafte Änderung in `/etc/selinux/config`.
        *   Kontext/Policies anpassen: `semanage`, `audit2allow`.

---

## 21. Port Knocking für SSH verwenden

*   **Warum:** Versteckt den SSH-Port, indem eine Sequenz von "Anklopfversuchen" auf andere Ports nötig ist, um den SSH-Port zu öffnen.
*   **Wie (Beispiel mit `knockd`):**
    1.  `knockd` installieren:
        ```bash
        sudo apt install knockd # Debian/Ubuntu
        ```
    2.  Konfigurieren (`/etc/knockd.conf`):
        ```ini
        [options]
            logfile = /var/log/knockd.log

        [openSSH]
            sequence    = 7000,8000,9000
            seq_timeout = 15
            command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
            tcpflags    = syn

        [closeSSH]
            sequence    = 9000,8000,7000
            seq_timeout = 15
            command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
            tcpflags    = syn
        ```
    3.  `knockd` starten/aktivieren:
        ```bash
        sudo systemctl enable --now knockd
        ```
    *   **Wichtig:** Firewall muss initial Port 22 blockieren!

---

## 22. Offene Ports limitieren

*   **Warum:** Reduziert die Angriffsfläche, da jeder offene Port ein potenzieller Einstiegspunkt ist.
*   **Wie:**
    1.  Offene Ports anzeigen:
        ```bash
        sudo ss -tulnp
        # oder
        sudo netstat -tulnp
        ```
    2.  Unnötige Dienste stoppen und deaktivieren (siehe Punkt 7).
    3.  Firewall nutzen, um nur notwendige Ports freizugeben (siehe Punkt 5).

---

## 23. Datei-Integritäts-Monitoring (FIM) verwenden

*   **Warum:** Erkennt unautorisierte Änderungen an kritischen Systemdateien.
*   **Wie (Beispiel mit AIDE):**
    1.  AIDE installieren:
        ```bash
        sudo apt install aide # Debian/Ubuntu
        ```
    2.  Datenbank initialisieren (dauert ggf. lange):
        ```bash
        sudo aideinit
        ```
    3.  Neue Datenbank aktivieren:
        ```bash
        sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
        ```
    4.  Regelmäßige Checks einrichten (z.B. per `cron`):
        ```bash
        sudo aide --check
        ```
    5.  Nach legitimen Änderungen die Datenbank aktualisieren: `sudo aide --update`

---

## 24. Rate Limiting implementieren

*   **Warum:** Schützt vor DoS-Angriffen und Brute-Force, indem die Anzahl der Anfragen/Logins von einer IP begrenzt wird.
*   **Wie:**
    *   **Mit `iptables` (Beispiel für SSH):**
        ```bash
        # Erstellt eine Liste 'SSH', fügt IP bei neuer Verbindung hinzu
        sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --set --name SSH

        # Dropt Verbindung, wenn > 4 Versuche in 60 Sek. von dieser IP kamen
        sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP
        ```
    *   **Mit `Fail2Ban`:** Parameter `findtime` und `maxretry` in der Konfiguration anpassen (siehe Punkt 6).
    *   **Webserver (Nginx):** `limit_req_zone` und `limit_req`.

---

## 25. Sensible Daten verschlüsseln

*   **Warum:** Schützt Daten bei Diebstahl oder unautorisiertem Zugriff (at Rest) und bei der Übertragung (in Transit).
*   **Wie:**
    *   **Data at Rest:**
        *   Festplattenverschlüsselung: LUKS (bei Installation oder `cryptsetup`).
        *   Home-Verzeichnis: `ecryptfs-utils`.
        *   Einzelne Dateien: GPG (`gpg`), `openssl`.
    *   **Data in Transit:**
        *   Immer TLS/SSL verwenden (HTTPS, SMTPS, IMAPS).
        *   VPNs (WireGuard, OpenVPN).
        *   SSH/SFTP/SCP statt Telnet/FTP.

---

## 26. DNS Security Extensions (DNSSEC) einrichten

*   **Warum:** Schützt vor DNS-Spoofing und Cache Poisoning durch Verifizierung von DNS-Antworten.
*   **Wie:**
    *   Hängt stark vom DNS-Provider oder der verwendeten DNS-Server-Software ab.
    *   **BIND:** In `named.conf.options` aktivieren:
        ```
        dnssec-enable yes;
        dnssec-validation auto;
        ```
    *   **Cloud Provider:** DNSSEC-Option in der Verwaltungsoberfläche aktivieren (z.B. AWS Route 53, Cloudflare).
    *   Domain-Registrar muss DNSSEC ebenfalls unterstützen.

---

## 27. Host-basiertes Intrusion Detection System (HIDS) nutzen

*   **Warum:** Überwacht den Server auf verdächtige Aktivitäten und alarmiert in Echtzeit.
*   **Wie (Beispiel OSSEC/Wazuh):**
    1.  Installieren (komplexer Prozess, folgt der jeweiligen Doku):
        *   Wazuh Agent auf dem Server, Wazuh Server zentral.
    2.  Konfigurieren, um Logs zu analysieren, Dateiintegrität zu prüfen, Rootkits zu erkennen etc.
    3.  Alarme konfigurieren.

---

## 28. Verschlüsselungsschlüssel und Zugangsdaten regelmäßig rotieren

*   **Warum:** Reduziert das Risiko, dass alte, potenziell kompromittierte Zugangsdaten weiterverwendet werden können.
*   **Wie:**
    *   Regelmäßige Passwortänderungen erzwingen (z.B. mit `chage`).
    *   SSH-Schlüssel periodisch erneuern.
    *   API-Schlüssel und Zertifikate vor Ablauf erneuern.
    *   Automatisierung mit Tools wie HashiCorp Vault oder Cloud KMS erwägen.

---

## 29. Prinzip der geringsten Rechte (PoLP) anwenden

*   **Warum:** Stellt sicher, dass Benutzer und Prozesse nur die Berechtigungen haben, die sie unbedingt benötigen.
*   **Wie:**
    *   `sudo`-Regeln spezifisch statt allgemein halten (siehe Punkt 19).
    *   Datenbankbenutzern nur benötigte Rechte geben (z.B. `GRANT SELECT, INSERT ON db.table TO 'user'@'host';`).
    *   Webserver-Prozesse unter eigenen, unprivilegierten Benutzern laufen lassen.
    *   Dateiberechtigungen restriktiv setzen (siehe Punkt 8).

---

## 30. Konfigurationsdrift überwachen

*   **Warum:** Verhindert, dass unbeabsichtigte oder unautorisierte Änderungen die Sicherheit schwächen.
*   **Wie:**
    *   Konfigurationsmanagement-Tools (Ansible, Puppet, Chef, SaltStack) verwenden, um den gewünschten Zustand zu definieren und durchzusetzen.
    *   Regelmäßige Audits mit Tools wie Lynis oder AIDE (siehe Punkt 15 & 23).
    *   Versionierung von Konfigurationsdateien (z.B. mit `etckeeper`).

---

## 31. Web Application Firewall (WAF) einrichten

*   **Warum:** Schützt Webanwendungen vor Angriffen wie SQL-Injection, XSS, CSRF.
*   **Wie (Beispiel mit ModSecurity für Apache/Nginx):**
    1.  ModSecurity installieren:
        ```bash
        # Apache (Debian/Ubuntu)
        sudo apt install libapache2-mod-security2
        # Nginx (Debian/Ubuntu) - oft als dynamisches Modul oder selbst kompiliert
        sudo apt install nginx-extras # Enthält oft ModSecurity
        ```
    2.  Core Rule Set (CRS) installieren (OWASP CRS empfohlen):
        ```bash
        sudo apt install modsecurity-crs
        # Oder manuell von GitHub herunterladen/konfigurieren
        ```
    3.  ModSecurity aktivieren (z.B. in Apache-Konfig):
        ```apache
        <IfModule security2_module>
            SecRuleEngine On
            # Include OWASP CRS rules
            IncludeOptional /usr/share/modsecurity-crs/crs-setup.conf
            IncludeOptional /usr/share/modsecurity-crs/rules/*.conf
        </IfModule>
        ```
    *   Regeln regelmäßig aktualisieren und anpassen (Tuning).

---

## 32. Application Sandboxing implementieren

*   **Warum:** Isoliert Anwendungen voneinander, um die Auswirkungen einer kompromittierten Anwendung zu begrenzen.
*   **Wie:**
    *   **Firejail:**
        ```bash
        sudo apt install firejail
        firejail <programmname> [argumente]
        ```
        *   Profile in `/etc/firejail/` anpassen.
    *   **AppArmor/SELinux:** Profile/Policies für Anwendungen erstellen/anpassen (siehe Punkt 20).
    *   **Container (Docker, Podman):** Anwendungen in Containern ausführen bietet inhärente Isolation.

---

## 33. 2FA für SSH mit Duo konfigurieren

*   **Warum:** Zusätzliche Sicherheitsebene für SSH durch einen zweiten Faktor (z.B. Push-Benachrichtigung).
*   **Wie:**
    1.  Duo Account erstellen und Anwendung (Unix Application) einrichten.
    2.  Duo PAM Modul installieren (z.B. `libpam-duo` oder von Duo).
    3.  Konfigurieren (`/etc/duo/pam_duo.conf`) mit Integration Key, Secret Key, API Host.
    4.  PAM für SSH anpassen (`/etc/pam.d/sshd`):
        ```
        auth required pam_duo.so
        ```
    5.  SSH-Konfiguration anpassen (`/etc/ssh/sshd_config`), falls nötig (`ChallengeResponseAuthentication yes`).
    6.  SSH-Dienst neu starten.

---

## 34. Regelmäßige Schwachstellen-Scans durchführen

*   **Warum:** Identifiziert proaktiv Sicherheitslücken in Server und Software.
*   **Wie:**
    *   Tools wie OpenVAS (GVM), Nessus Essentials (kostenlos für kleine Netzwerke) oder kommerzielle Scanner verwenden.
    *   Beispiel (Installation von OpenVAS/GVM ist komplex): Scans planen und Ergebnisse analysieren/beheben.
    *   Auch Webserver-Scanner wie Nikto oder OWASP ZAP nutzen, falls Webanwendungen laufen.
        ```bash
        sudo apt install nikto
        nikto -h http://your-server-ip
        ```

---

## 35. Data Loss Prevention (DLP) Maßnahmen umsetzen

*   **Warum:** Schützt sensible Informationen vor unautorisiertem Zugriff und Leaks.
*   **Wie:**
    *   FIM-Tools (AIDE) zur Überwachung von Zugriffen/Änderungen (siehe Punkt 23).
    *   Datenverschlüsselung (at Rest / in Transit) (siehe Punkt 25).
    *   Strikte Zugriffsrechte und ACLs (siehe Punkte 8, 19, 40).
    *   Netzwerk-DLP-Lösungen (komplexer) oder Endpunkt-DLP (falls zutreffend).

---

## 36. Unveränderliche Backups und Snapshots verwenden

*   **Warum:** Stellt sicher, dass Backups nicht manipuliert oder gelöscht werden können (z.B. durch Ransomware).
*   **Wie:**
    *   Backup-Lösungen nutzen, die Immutability unterstützen (viele Cloud-Anbieter, manche Backup-Software wie BorgBackup mit append-only Repositories).
    *   Regelmäßige Snapshots auf Dateisystem- (ZFS, Btrfs) oder Virtualisierungsebene.
    *   Air-Gapped Backups (physisch getrennt).

---

## 37. Erweitertes Auditing mit Auditbeat/Filebeat konfigurieren

*   **Warum:** Detaillierte Überwachung von Systemaufrufen, Dateiintegrität, Anmeldeversuchen etc., oft in Kombination mit zentralem Logging.
*   **Wie (Elastic Stack):**
    1.  Auditbeat und/oder Filebeat installieren:
        ```bash
        # Download von Elastic Website oder über deren Repository
        sudo apt install auditbeat filebeat
        ```
    2.  Konfigurieren (`/etc/auditbeat/auditbeat.yml`, `/etc/filebeat/filebeat.yml`), um Daten an Elasticsearch/Logstash zu senden.
    3.  Auditbeat-Module aktivieren (z.B. file_integrity, system).
    4.  Dienste starten/aktivieren.

---

## 38. Remote Logging einrichten

*   **Warum:** Sichert Logdaten, selbst wenn der Server kompromittiert wird (Logs sind auf einem anderen System).
*   **Wie (mit `rsyslog`):**
    1.  Auf dem sendenden Server (`/etc/rsyslog.conf` oder in `/etc/rsyslog.d/`):
        ```
        # Sende alle Logs an den Remote-Server via UDP
        *.* @remote_log_server_ip:514

        # Oder via TCP
        # *.* @@remote_log_server_ip:514
        ```
    2.  Auf dem empfangenden Server `rsyslog` konfigurieren, um Remote-Logs anzunehmen (z.B. in `/etc/rsyslog.conf` UDP/TCP-Input aktivieren).
    3.  Firewalls anpassen.
    4.  `rsyslog`-Dienst auf beiden Servern neu starten.

---

## 39. Regelmäßige Penetrationstests durchführen

*   **Warum:** Simuliert Angriffe, um Schwachstellen aufzudecken, die durch automatisierte Scans evtl. übersehen werden.
*   **Wie:**
    *   Externe, qualifizierte Pentester beauftragen.
    *   Interne Tests mit Tools wie Metasploit Framework, Nmap, Nikto, SQLMap etc. durchführen (erfordert Know-how!).
    *   Ergebnisse analysieren und Schwachstellen beheben.

---

## 40. Access Control Lists (ACLs) für feingranulare Berechtigungen nutzen

*   **Warum:** Erlaubt spezifischere Berechtigungen als traditionelle Unix-Permissions (rwx für user/group/other).
*   **Wie:**
    1.  Prüfen, ob ACLs unterstützt/aktiviert sind (meist Standard bei modernen FS): `mount | grep acl`
    2.  ACL setzen:
        ```bash
        # Benutzer 'bob' Lese-/Schreibzugriff auf Datei geben
        sudo setfacl -m u:bob:rw /pfad/zur/datei

        # Gruppe 'webdev' Lesezugriff auf Verzeichnis (und neue Dateien darin) geben
        sudo setfacl -m g:webdev:rX /pfad/zum/verzeichnis
        sudo setfacl -d -m g:webdev:rX /pfad/zum/verzeichnis
        ```
    3.  ACLs anzeigen:
        ```bash
        getfacl /pfad/zur/datei
        ```

---

## 41. Bastion Hosts (Jump Hosts) für sicheren Serverzugriff verwenden

*   **Warum:** Zentralisiert und kontrolliert den Zugriff auf interne Server, fügt eine zusätzliche Sicherheitsschicht und Audit-Möglichkeit hinzu.
*   **Wie:**
    *   Einen dedizierten, stark abgesicherten Server (Bastion Host) im öffentlichen Netz oder DMZ aufsetzen.
    *   Firewall-Regeln so konfigurieren, dass SSH-Zugriff auf interne Server *nur* vom Bastion Host erlaubt ist.
    *   Auf dem Bastion Host: Strikte Authentifizierung (Keys, MFA), minimale Software, detailliertes Logging.
    *   Verbindung zum Zielserver über den Bastion Host: `ssh -J user@bastion_ip user@ziel_ip`

---

## 42. Datenbankzugriff härten

*   **Warum:** Datenbanken enthalten oft sensible Daten und sind häufige Angriffsziele.
*   **Wie:**
    *   Netzwerkzugriff beschränken (z.B. nur von Applikationsserver-IPs erlauben, in `my.cnf`, `pg_hba.conf` etc.).
    *   Starke, einzigartige Passwörter für DB-Benutzer verwenden.
    *   Prinzip der geringsten Rechte anwenden (siehe Punkt 29).
    *   Verschlüsselung für Daten at Rest und in Transit nutzen.
    *   Standard-Benutzernamen/-Ports ändern (optional).
    *   Regelmäßige Updates der DB-Software.

---

## 43. Logs regelmäßig überprüfen und analysieren

*   **Warum:** Nur geloggte, aber nie geprüfte Events sind nutzlos. Regelmäßige Analyse deckt verdächtige Aktivitäten auf.
*   **Wie:**
    *   Zentrale Logging-Tools (ELK, Graylog, Splunk) nutzen (siehe Punkt 9, 38, 56).
    *   Automatisierte Alarme für kritische Events einrichten (z.B. mehrfache fehlgeschlagene Logins, `sudo`-Nutzung, Fehler).
    *   Manuelle Stichproben wichtiger Logs (`/var/log/auth.log`, `/var/log/syslog`, App-Logs).
    *   Tools wie `logwatch` für tägliche Zusammenfassungen verwenden.

---

## 44. Festplattenpartitionen verschlüsseln

*   **Warum:** Schützt Daten bei physischem Diebstahl oder unbefugtem Zugriff auf die Hardware.
*   **Wie (mit LUKS):**
    *   **Bei Neuinstallation:** Option für Vollverschlüsselung wählen.
    *   **Nachträglich (Beispiel für separate Partition `/dev/sdxY`):**
        1.  LUKS-Container erstellen (ACHTUNG: Daten gehen verloren!):
            ```bash
            sudo cryptsetup luksFormat /dev/sdxY
            ```
        2.  Container öffnen (entschlüsseln und Device Mapping erstellen):
            ```bash
            sudo cryptsetup luksOpen /dev/sdxY meine_verschluesselte_partition
            ```
        3.  Dateisystem erstellen:
            ```bash
            sudo mkfs.ext4 /dev/mapper/meine_verschluesselte_partition
            ```
        4.  Mounten und in `/etc/fstab` eintragen (mit `/etc/crypttab` für automatisches Entsperren beim Boot).

---

## 45. Zero-Trust-Architekturprinzipien umsetzen

*   **Warum:** Geht davon aus, dass kein Akteur (Benutzer, Gerät, Netzwerk) vertrauenswürdig ist; verlangt strikte Verifizierung für jeden Zugriff.
*   **Wie (Prinzipien, keine einzelnen Befehle):**
    *   Strikte Identitätsprüfung (MFA überall).
    *   Mikrosegmentierung (feingranulare Netzwerkisolation).
    *   Least Privilege Access (PoLP) rigoros durchsetzen.
    *   Kontinuierliche Überwachung und Validierung von Zugriffen.
    *   Policy Engines (z.B. Open Policy Agent) für Zugriffsentscheidungen nutzen.

---

## 46. Honeypot-System zur Erkennung einsetzen

*   **Warum:** Lockt Angreifer in eine kontrollierte Falle, um deren Methoden zu studieren und Angriffe auf produktive Systeme frühzeitig zu erkennen.
*   **Wie (Beispiel mit `Cowrie` für SSH/Telnet):**
    1.  Honeypot-Software installieren (z.B. Cowrie, Dionaea). Läuft oft am besten in Docker oder eigener VM.
        ```bash
        # Installation variiert stark, oft via Docker oder Python/pip
        ```
    2.  Konfigurieren, um wie ein verwundbares System auszusehen.
    3.  Auf einem separaten Netzwerksegment platzieren.
    4.  Logs des Honeypots überwachen.

---

## 47. Serverhärtung mit CIS Benchmarks implementieren

*   **Warum:** Bietet einen industrie-anerkannten Standard für sichere Konfigurationen.
*   **Wie:**
    1.  Passenden CIS Benchmark für das Betriebssystem herunterladen (Registrierung oft erforderlich).
    2.  Manuell die Empfehlungen durchgehen und umsetzen.
    3.  Automatisierte Tools zur Prüfung verwenden:
        *   `Lynis` prüft viele CIS-Kontrollen (`sudo lynis audit system --profile /pfad/zum/cis-profile`).
        *   `OpenSCAP` kann ebenfalls zur Prüfung gegen Sicherheitsrichtlinien (inkl. CIS) verwendet werden.
        ```bash
        sudo apt install openscap-scanner libopenscap8 # Debian/Ubuntu
        # Download OVAL/XCCDF-Dateien für CIS
        # sudo oscap xccdf eval --profile cis_profile.xml scap_definition.xml
        ```

---

## 48. Just-In-Time (JIT) Zugriffskontrollen verwenden

*   **Warum:** Reduziert das Risiko, indem temporärer Zugriff nur bei Bedarf und für begrenzte Zeit gewährt wird.
*   **Wie:**
    *   Oft über IAM-Systeme in Cloud-Umgebungen (AWS IAM, Azure PIM).
    *   Tools wie Teleport, HashiCorp Boundary oder selbst entwickelte Lösungen mit temporären SSH-Zertifikaten/Credentials.
    *   Workflow: Anforderung -> Genehmigung -> Temporärer Zugriff -> Automatischer Entzug.

---

## 49. Endpoint Detection and Response (EDR) Tools implementieren

*   **Warum:** Bietet erweiterte Bedrohungserkennung, Verhaltensanalyse und Incident Response-Fähigkeiten auf dem Server (Endpunkt).
*   **Wie:**
    *   Kommerzielle EDR-Lösungen (CrowdStrike Falcon, SentinelOne, Carbon Black) oder Open-Source-Alternativen (Wazuh kann EDR-Funktionen bieten).
    *   Agent auf dem Server installieren.
    *   Policies konfigurieren (was wird überwacht, wie wird reagiert).
    *   Integration mit SIEM (siehe Punkt 56).

---

## 50. Hardware Security Modules (HSMs) für Schlüsselmanagement verwenden

*   **Warum:** Bietet manipulationssichere Speicherung und Verwaltung kryptographischer Schlüssel für höchste Sicherheitsanforderungen.
*   **Wie:**
    *   Physisches HSM-Gerät oder Cloud-HSM-Dienst (AWS CloudHSM, Azure Dedicated HSM) beschaffen/mieten.
    *   Anwendungen konfigurieren, um das HSM für Schlüsselgenerierung, -speicherung und kryptographische Operationen zu nutzen (z.B. über PKCS#11-Schnittstelle).
    *   Sehr teuer und komplex; nur bei extrem hohen Sicherheitsanforderungen sinnvoll.

---

## 51. Immutable Infrastructure Prinzipien anwenden

*   **Warum:** Verhindert Konfigurationsdrift und unbemerkte Änderungen, indem Server bei Updates/Änderungen komplett ersetzt statt modifiziert werden.
*   **Wie:**
    *   Images (z.B. AMIs, Docker-Images) mit Tools wie Packer, Dockerfile erstellen.
    *   Infrastruktur als Code (Terraform, CloudFormation, Pulumi) verwenden, um Deployments zu automatisieren.
    *   Bei Updates: Neue Instanzen/Container aus dem neuen Image starten, alte terminieren (Blue/Green Deployment, Canary Releases).

---

## 52. Regelmäßige Compliance-Audits durchführen

*   **Warum:** Stellt sicher, dass der Server Industriestandards und gesetzlichen Vorgaben entspricht (z.B. DSGVO/GDPR, HIPAA, PCI-DSS).
*   **Wie:**
    *   Anforderungen der relevanten Standards verstehen.
    *   Tools wie `OpenSCAP` (siehe Punkt 47), `Lynis` oder kommerzielle Audit-Tools nutzen.
    *   Manuelle Überprüfung von Konfigurationen, Logs, Prozessen.
    *   Dokumentation der Ergebnisse und Behebungsmaßnahmen.

---

## 53. Disaster Recovery Plan (DRP) erstellen

*   **Warum:** Ermöglicht schnelle Wiederherstellung und Betriebskontinuität nach schwerwiegenden Ausfällen oder Katastrophen.
*   **Wie:**
    *   Kritische Systeme und Daten identifizieren (Recovery Point Objective - RPO, Recovery Time Objective - RTO definieren).
    *   Regelmäßige, getestete Backups (siehe Punkt 13), idealerweise extern/offsite.
    *   Plan für die Wiederherstellung von Systemen und Daten dokumentieren.
    *   Regelmäßige Tests des DRP durchführen (Simulation).

---

## 54. Kernel mit Grsecurity härten (Hinweis beachten)

*   **Warum:** Bietet erweiterte Kernel-Sicherheitsfeatures gegen Exploits und zur Zugriffskontrolle.
*   **Wie:**
    *   Grsecurity ist kommerziell und erfordert ein Abonnement.
    *   Patches herunterladen, auf Kernel-Source anwenden, Kernel neu kompilieren und installieren.
    *   Konfigurieren (sehr komplex).
    *   **Alternative:** Standard-Kernel-Härtungsoptionen nutzen (siehe Punkt 12), SELinux/AppArmor (Punkt 20).

---

## 55. Speicherschutz mit ExecShield aktivieren (primär ältere Systeme/RHEL-basiert)

*   **Warum:** Schützt vor Buffer Overflows, indem Speichersegmente als nicht ausführbar markiert werden (NX-Bit/XD-Bit).
*   **Wie:**
    *   Auf vielen modernen Systemen ist dies (oder Äquivalente wie DEP) standardmäßig aktiv.
    *   Prüfen/Einstellen über `sysctl` (falls vom Kernel unterstützt):
        ```bash
        # Prüfen (1 = aktiv, 0 = inaktiv)
        sysctl kernel.exec-shield

        # Aktivieren (falls möglich und nicht standard)
        # In /etc/sysctl.conf eintragen:
        # kernel.exec-shield = 1
        # sudo sysctl -p
        ```
    *   ASLR (Address Space Layout Randomization) ist oft wichtiger und standardmäßig aktiv (`kernel.randomize_va_space=2`).

---

## 56. Security Information and Event Management (SIEM) einrichten

*   **Warum:** Sammelt und korreliert Logdaten aus verschiedenen Quellen, um Sicherheitsvorfälle zentral zu erkennen und zu analysieren.
*   **Wie:**
    *   SIEM-Lösung wählen: Open Source (ELK Stack + Security Onion/Wazuh, Graylog) oder kommerziell (Splunk, QRadar, LogRhythm).
    *   Alle relevanten Logquellen (Server, Firewalls, Anwendungen, HIDS/NIDS) an das SIEM senden (siehe Punkt 38).
    *   Korrelationsregeln definieren, um verdächtige Muster zu erkennen.
    *   Dashboards und Alarme einrichten.

---

## 57. Role-Based Access Control (RBAC) für Anwendungen einschränken

*   **Warum:** Setzt Least Privilege auf Anwendungsebene durch, indem Berechtigungen an Rollen statt an einzelne Benutzer gebunden werden.
*   **Wie:**
    *   Abhängig von der Anwendung. Viele moderne Anwendungen/Frameworks haben eingebaute RBAC-Systeme.
    *   Rollen definieren (z.B. `admin`, `editor`, `viewer`).
    *   Jeder Rolle spezifische Berechtigungen zuweisen.
    *   Benutzer den entsprechenden Rollen zuordnen.
    *   Regelmäßige Überprüfung der Rollen und Berechtigungen.

---

## 58. Data Retention Policy erstellen

*   **Warum:** Definiert, wie lange Daten aufbewahrt werden müssen/dürfen; reduziert Risiken und Kosten durch unnötige Datenhaltung.
*   **Wie:**
    *   Gesetzliche und geschäftliche Anforderungen an die Datenaufbewahrung ermitteln.
    *   Richtlinie erstellen, die Aufbewahrungsfristen für verschiedene Datentypen festlegt.
    *   Automatisierte Löschprozesse implementieren (z.B. `cron` jobs, Lifecycle Policies in Cloud Storage).
    *   Sichere Löschmethoden verwenden (z.B. `shred`, `wipe`).

---

## 59. Honeytokens zur Erkennung von unbefugtem Zugriff einrichten

*   **Warum:** Köderdaten (z.B. gefälschte API-Keys, DB-Einträge), die bei Zugriff einen Alarm auslösen und so unbemerkten Zugriff aufdecken.
*   **Wie:**
    *   Attraktive, aber gefälschte Daten/Credentials erstellen (z.B. Datei `aws_credentials.txt`, DB-Eintrag `customer_ceo_private`).
    *   Überwachung einrichten, die alarmiert, wenn genau diese Daten gelesen/verwendet werden (z.B. über `auditd`, FIM, SIEM).
    *   Platzierung an Orten, wo ein Angreifer suchen würde.
