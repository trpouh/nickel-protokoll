Veeam läuft auf einem Windows Server 2016 mit einer Topologie, ersichtlich in Abbildung \ref{veeam-grob}.

\begin{figure}[!htb]
\centering
\includegraphics{./images/veeam_grob.jpg}
\caption{Veeam Aufbau}\label{veeam-grob}
\end{figure}

# Installation

1. Software von Veeam herunterladen von der [Veeam Download Seite](www.veeam.com/downloads.html)

2. Software als ISO in eine VM einlegen

3. Setup.exe in der VM ausführen und im Wizard auf Install clicken

4. Lizenzeinstellungen akzeptieren und ohne Lizenzdatei fortfahren

5. Veeam Backup & Replication sowie die zugehörige Console im Wizard auswählen

6. fehlende Software automatisch nachinstallieren lassen

7. Installationsdetails auswählen

   | Einstellung             | Wert                                             |
   | ----------------------- | ------------------------------------------------ |
   | Installationsordner     | C:\\Program Files\\Veeam\\Backup and Replication |
   | vPower Cache            | C:\\ProgramData\\Veeam\\Backup\\NfsDatastore     |
   | Guest Catalog           | C:\\VBRCatalog                                   |
   | Catalog Service Port    | 9393                                             |
   | Service Port            | 9392                                             |
   | Secure Connections Port | 9401                                             |
   | Service Account         | LOCAL SYSTEM                                     |
   | SQL Server              | LOCALHOST\\VEEAMSQL2012                          |
   | Datenbank Name          | VeeamBackup                                      |

8.  Installation abschließen

# Hinzufügen des Hypervisors

1. Veeam Dienste starten
2. Veeam Backup and Replication Console öffnen
3. Neuen VMware Server Wizard starten
   1. Backup Infrastruktur öffnen
   2. im Tab Verwaltete Server neuen Server hinzufügen
   3. VMware Vsphere auswählen
   4. Server-Details eintragen
4. Überprüfung
   1. Im Inventar sind jetzt alle Ressourcenpools und VMs sichtbar

# Backup via VeeamZIP

VeeamZIP ist ein Feature, dass VM-Backups in Form von ZIP-Archiven erlaubt. Dafür muss im Inventar-Menü die Ziel-VM ausgewählt werden und im anschließenden Menü die entsprechenden Einstellungen anpassen (siehe Abbildung \ref{Veeam-zip}).

\begin{figure}[!htb]
\centering
\includegraphics{./images/veeam_zip.jpg}
\caption{Veeam ZIP}\label{Veeam-zip}
\end{figure}

# Restore von VeeamZIP

Um eine VeeamZIP wiederherzustellen muss zunächst oben "Restore" geklickt und dann das VeeamZIP-Archiv ausgewählt werden. Anschließend kann man noch auswählen, wohin die Maschine exportiert werden soll, wie in Abbildung \ref{Veeam_restore} sehen kann.

\begin{figure}[!htb]
\centering
\includegraphics{./images/veeam_restore.jpg}
\caption{VeeamZIP restore}\label{Veeam-restore}
\end{figure}

# Lessons Learned

Veeam wird oft als DIE de facto VMware und Hyper-V Backup Lösung angesehen. Die genialen Aspekte, wie etwa differentielles oder inkrimentelles Backup sind dabei in der gratis Testversion leider nicht verfügbar. VeeamZIP hat allerdings gut funktioniert. Zusätzlich unterstützt Veeam thin und thick Provisioning, was in Praxis Situationen außerordentliche Flexibilität erlaubt. Außerdem erlaubt Veeams auch in anderen Hinsichten wahren Enterprise Support. So kann es beispielsweise mit Copyjobs Backups auf einen externen Speichercluster schreiben. Alles in allem macht das Veeam genial für Enterprises, für den privaten Gebrauch scheinen allerdings Backups als ausreichend.