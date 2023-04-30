| ![Views](https://img.shields.io/endpoint?color=green&label=Views&logo=Debian&logoColor=red&style=plastic&url=https%3A%2F%2Fhits.dwyl.com%2FTomfromBerlin%2FDebian-Pihole-Unbound) | ![Unique Viewers](https://img.shields.io/endpoint?color=green&label=Unique%20Viewers&logo=Raspberry%20Pi&logoColor=pink&style=plastic&url=https%3A%2F%2Fhits.dwyl.com%2FTomfromBerlin%2FDebian-Pihole-Unbound.svg%3Fshow%3Dunique) | ![DEBIAN](https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white) | ![RaspberryPi](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=for-the-badge&logo=Raspberry%20Pi&logoColor=white) | ![FRITZ!box 7490](https://user-images.githubusercontent.com/123265893/219904891-ab50d92a-ad69-480a-adc3-dd37b8a21534.svg) | [English Version](/../../../../TomfromBerlin/Debian-Pihole-Unbound/blob/main/README.md)|
 |-|-|-|-|-|-|
 
# _Debian (Bullseye) mit Pi-hole & Unbound & Fritzbox 7490_ 
Dies ist ein Versuch zu zeigen, wie Pi-hole und Unbound dazu gebracht werden können, unter Debian (Bullseye) zu arbeiten. Hol dir eine Tasse Kaffee oder Tee, es gibt viel zu lesen.

Obwohl diese Anleitung hier in deutscher Sprache vorliegt, werden mitunter Verlinkungen zu Seiten in englischer Sprache angegeben. Die Seite von Pi-hole sei hier beispielhaft genannt. Basiskenntnisse in englischer Sprache sind also durchaus angebracht.

| Haftungsausschluss: Der hier beschriebene Weg hat bei mir funktioniert. Ob es bei dir auch funktioniert kann ich nicht sagen. Aber vielleicht hilft dir diese Beschreibung, den richtigen Weg für dich zu finden. Es gibt jedoch keinerlei Garantie auf irgendwas. Viel Glück damit. |
|:-|

Eine weitere Beschreibung, wie das alles funktioniert, ist hier zu finden: <https://docs.pi-hole.net/guides/dns/unbound/#>

## _Warum noch ein Tutorial zum Einrichten von Pihole und Unbound_

Nun, obwohl es viele Tutorials gibt, hat mich davon keines von einem frisch installierten Betriebssystem zu einem funktionierenden lokalen Domain Name Server geführt. Ich musste das Internet durchforsten und Informationen und Hinweise sammeln. Dann musste ich versuchen, was funktioniert. Einiges war veraltet, anderes einfach falsch. Aber schließlich habe ich es geschafft und jetzt habe ich einen lokalen Domain Name Server und Pi-Hole bewacht mein lokales Netzwerk mit allen Geräten, die mit diesem Netzwerk verbunden sind.

Eines habe ich auf die harte Tour gelernt: Sobald es ein laufendes System gibt, mach ein Backup. Das Betriebssystem ist installiert und aktualisiert? Erstelle ein Backup. Pi-hole und Unbound sind installiert und es funktioniert? Backup! Der Raspberry Pi ist erfolgreich ins Netzwerk integriert? Backup und Screenshots der Routerkonfiguration. Jetzt kann ich einfach einen Rollback durchführen und habe ruck-zuck meinen Domain Name Server wieder lauffähig ohne lästiges Ausprobieren, bis er richtig funktioniert.

## _Systemanforderungen_ 

Man benötigt ein lokales Netzwerk (logisch), einen Router als Gateway zum Internet (offensichtlich), einen Raspberry Pi und einen zweiten PC. Den zweiten PC bezeichne ich hier der Einfachheit halber als Workstation. Außerdem entweder eine externe Festplatte oder eine SD-Karte. Bei der Verwendung einer SD-Karte muss sichergestellt werden, dass von der Workstation darauf geschrieben werden kann. Ich empfehle jedoch die Verwendung einer externen Festplatte, da SD-Karten eher zum Speichern von Daten als zum Betrieb als Systemlaufwerk in einem Computer konzipiert sind. Beim Betrieb mit einer SD-Karte sind Ausfälle daher wahrscheinlicher.

Das Betriebssystem auf der Workstation kann entweder Linux, Windows oder iOS sein. Das spielt kaum eien Rolle. Die Workstation und auf jeden Fall der Raspberry Pi sollten via **Netzwerkkabel** mit dem Netzwerk verbunden sein. Der Raspberry Pi sollte _nicht_ über WLAN in das Heimnetz integriert werden, da die Verbindung möglicherweise instabil ist.

Nicht zuletzt wird auch ein Image des neuesten Raspberry Pi OS oder zumindest das Programm rpi-imager benötigt. Wenn das Image herunter geladen wurde, kann dieses z.B. mit balenaEtcher auf das Laufwerk geschrieben werden.

Wenn der Pi zum ersten Mal mit dem neuen Betriebssystem startet, wird der Bootvorgang etwas länger dauern. Später geht das schneller. An dieser Stelle ist ein angeschlossener Monitor genauso aus wie eine Tastatur mehr als nur hilfreich. Später wird der Zugriff über ssh erfolgen, aber das steht noch nicht zur Verfügung. Die Fragen zu bestimmten Einstellungen (ssh- und VNC-Aktivierung, Länder- und Tastatureinstellungen) sollten entsprechend beantwortet werden. ssh sollte auf jeden Fall aktiviert werden. Sobald ssh aktiviert ist, kann über dieses Protokoll auf den Pi zugriffen werden. Das Pi-hole-Interface wird im Webbrowser via HTML aufgerufen.

Jetzt wäre auch ein guter Zeitpunkt, die Lokalisierung vorzunehmen. Entweder mit raspi-config (Kommandozeile oder Desktop-Umbebung) die Einstellungen vornhemen oder auf der Kommandozeile `dpkg-reconfigure locales` eingeben und `de_DE.UTF-8 UTF-8` als default angeben. Das kann aber später auch per ssh erledigt werden.

Optional: Mit `apt install task-german` können noch diverse deutsche man-pages installiert werden.

Übrigens, wenn die Installation mit dem rpi-imager durchgeführt wird, wird bereits vor der Installation gefragt, ob ssh aktiviert werden soll. Auch Fragen nach Länder- und Tastatureinstellungen werden gestellt. Zu diesem Zeitpunkt kann man sich wohl die Beantwortung sparen. In meinem Fall zumindest war es notwendig, diese Einstellungen nach dem ersten Booten nochmals vorzunehmen.

Die folgenden Schritte sind _headless_ via ssh möglich.

Nach der Installation sollte das System aktualisiert und eventuell der [Midnight Commander](/../../../../MidnightCommander/mc) installiert werden:

```
$apt update
$apt upgrade
$apt install mc
```

Der Midnight Commander kann später verwendet werden, um durch das Dateisystem zu navigieren. Meiner Meinung nach ist dies übersichtlicher und die Navigation etwas schneller (z.B. `Ctrl-Bild hoch`/`Strg-Bild hoch` springt in das nächsthöhere Verzeichnis - wobei das Root-Verzeichnis das höchste ist. Zusätzlich steht mit dem Hotkey F3 ein Dateibetrachter zur Verfügung und mit F4 ein Editor.

Fast alle folgenden Operationen erfordern erhöhte Rechte, d.h. Sie müssen root sein.

Der Midnight Commander kann mit `sudo mc` gestartet werden, um zu verhindern, dass jedes Mal `sudo` eingegeben werden muss, um eine Datei zu bearbeiten. Aber Vorsicht ist geboten. In diesem Modus kann versehentlich viel Unheil angerichtet werden.

Das Tastenkürzel `Strg-o` versetzt Midnight Commander in den Hintergrund, erneutes Drücken bringt ihn wieder in den Vordergrund. Dies ist nützlich, wenn die Konsole verwendet wird und die Ausgabe eines Befehls sichtbar sein soll, z.B. bei dem Befehl `$systemctl status hciuart.service`, den wir jetzt ausführen.

## Bluetooth Service kann nicht gestartet werden

|❗| Update 30.April 2023 | Nach der Systemaktualisierung hatte ich die gleichen Probleme mit Bluetooth wie unten beschrieben. Also habe ich versucht, bluetooth.service neu zu laden, nachdem die Boot-Sequenz beendet war, was nicht gelang. Ich beschloss, den hinzugefügten Eintrag in /etc/boot/config.txt `dtparam=krnbt` auszukommentieren. Nun, das System bootete ohne jegliche Beschwerden bezüglich Bluetooth. Ich werde das weiter beobachten und dieses Tutorial bei Bedarf aktualisieren. | OS: RaspiOS - Distribution: Debian GNU/Linux 11 (bullseye) 11 - Kernel Release: 6.1.21-v8+ - Kernel Version: #1642 SMP PREEMPT Mon Apr  3 17:24:16 BST 2023 - Architecture: aarch64 |❗|
|-|-|:-|:-|-|

Auf einem frisch installierten Debian (bullseye) System haben wir unter Umständen das Problem, dass hciuart.service nicht gestartet werden kann. Der Befehl `$systemctl status hciuart.service` (als root) zeigt dann dies:

```
● hciuart.service - Configure Bluetooth Modems connected by UART
     Loaded: loaded (/lib/systemd/system/hciuart.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Sat 2023-02-11 12:08:05 CET; 41min ago
    Process: 422 ExecStart=/usr/bin/btuart (code=exited, status=1/FAILURE)
        CPU: 191ms

Feb 11 12:07:18 raspberrypi systemd[1]: Starting Configure Bluetooth Modems connected by UART...
Feb 11 12:08:06 raspberrypi btuart[481]: Initialization timed out.
Feb 11 12:08:06 raspberrypi btuart[481]: bcm43xx_init
Feb 11 12:08:06 raspberrypi btuart[481]: Flash firmware /lib/firmware/brcm/BCM4345C0.hcd
Feb 11 12:08:05 raspberrypi systemd[1]: hciuart.service: Control process exited, code=exited, status=1/FAILURE
Feb 11 12:08:05 raspberrypi systemd[1]: hciuart.service: Failed with result 'exit-code'.
Feb 11 12:08:05 raspberrypi systemd[1]: Failed to start Configure Bluetooth Modems connected by UART.
```

Du kannst versuchen, dies mit den folgenden Schritten zu beheben (als root).

1) Ohne Midnight Commander:
     1) den Befehl `$nano /etc/systemd/system/bluetooth.target.wants/bluetooth.service` auf der Kommandozeile eingeben
     2) die Zeile `ExecStart=/usr/lib/bluetooth/bluetoothd` nach `ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap` ändern
     3) die Datei mit `Ctrl-o`/`Strg-o` speichern und nano mit `Ctrl-x `/`Strg-x` beenden

2) Mit Midnight Commander:
     1) `sudo mc`
     2)  zum Verzeichnis `/etc/systemd/system/bluetooth.target.wants/` navigieren, die Datei `bluetooth.service` markieren und F4 drücken
     4)  die Zeile `ExecStart=/usr/lib/bluetooth/bluetoothd` nach `ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap` ändern
     5)  F10 drücken und die Abfrage nach dem Speichern der Datei mit `Ja`/`Yes` beantworten

3)  `Strg-o` drücken um den Midnight Commander in den Hintergrund zu schieben
4)  Systemd neu laden: `$systemctl daemon-reload`
5)  Bluetooth neu starten: `$service bluetooth restart`

Die Ausgabe von `$systemctl status hciuart.service` vermittelt jetzt, dass alles okay zu sein scheint.

```
    bluetooth.service - Bluetooth service
       Loaded: loaded (/lib/systemd/system/bluetooth.service; enabled)
       Active: active (running) since Sat 2016-04-30 10:38:46 UTC; 6s ago
         Docs: man:bluetoothd(8)
     Main PID: 12775 (bluetoothd)
       Status: "Running"
       CGroup: /system.slice/bluetooth.service
               └─12775 /usr/lib/bluetooth/bluetoothd --noplugin=sap
```

Wenn die Systemdatei bluetooth.service nicht verändert werden soll, kann eine .service.d-Überschreibung verwendet werden:

1) neues Verzeichnis erstellen mit `$mkdir /etc/systemd/system/bluetooth.service.d/`
2) eine neue Datei anlegen mit: `$touch /etc/systemd/system/bluetooth.service.d/01-disable-sap-plugin.conf`
3) die Datei bearbeiten mit: `$nano /etc/systemd/system/bluetooth.service.d/01-disable-sap-plugin.conf` und die folgenden drei Zeilen in die Datei einfügen (oder den Midnight Commander nach obigem Schmea verwenden):

    ```
    [Service]
    ExecStart=
    ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap
    ```

4) Speichern mit `Ctrl-o`/`Strg-o`; nano mit `Ctrl-x`/`Strg-x` beenden (im Editor des Midnght Commanders F10 drücken und mit `ja`/`Yes` antworten.

_(Einfache) Erklärung: der Wert von ExecStart= wird mit der Anweisung in der zweiten Zeile gelöscht, sodass er neu geschrieben werden kann, anstatt ihn anzuhängen. Einige SystemD-Einstellungen verhalten sich wie eine angehängte Liste, wenn sie mehrmals angegeben werden. Die dritte Zeile definiert den neuen Wert von ExecStart=_

Dann wie oben die folgenden Befehle in die Kommandozeile eingeben:

```
$systemctl daemon-reload
$systemctl restart bluetooth.service
```

(Quelle: <https://raspberrypi.stackexchange.com/questions/40839/sap-error-on-bluetooth-service-status>)

Jetzt sollte hciuart.service betriebsbereit sein, jedoch ist das leider nur die halbe Wahrheit, denn der nächste Neustart steht bevor und das selbe Problem wird wieder auftauchen: hciuart.service kann nicht gestartet werden. Das eigentliche Problem liegt darin, dass das BT-Modem zu spät initialisiert wird. Aber zumindest ist eine Fehlermeldung weg, also ist die Arbeit nicht ganz umsonst.

|‼️| Was tatsächlich hilft, ist ein neuer Eintrag in der Datei _/boot/config.txt_: `dtparam=krnbt`, der dem Kernel die Aufgabe gibt, das BT-Modem zu initialisieren. Es gibt bereits Einträge, die mit dtparam= beginnen, ich habe den neuen dort hinzugefügt und es scheint zu helfen. Dies wird wahrscheinlich in Zukunft der Standard werden, wie ich gehört habe. An dieser Stelle ein großes Dankeschön an [Phil Elwell](/../../../../pelwell) für den [entscheidenden Hinweis](https://github.com/RPi-Distro/pi-bluetooth/issues/25#issuecomment-1426768853). |‼️|
|:-:|:-|:-:|

## _Pi-hole und Unbound installieren_

Nachdem wir nun ein frisch eingerichtetes System und die oben genannten Fehler behoben oder ignoriert haben, können wir uns nun der eigentlichen Aufgabe zuwenden: der Installation von Pi-hole und Unbound. Aber vorher sollten wir

### _eine statische IP-Adresse für den Raspberry Pi festlegen_

Dies geschieht über das Router-Interface, indem in der Adressleiste des bevorzugten Browsers http://fritz.box oder http://ip-adresse-des-routers eingeben wird.

Dann zu folgendem Bildschirm navigieren

![fritzbox_network_connections](https://user-images.githubusercontent.com/123265893/218282027-874f4508-c003-4fc8-ab3f-f162ed180dd1.PNG) 

den Raspberry Pi suchen und auf das Stiftsymbol auf der rechten Seite klicken. Die nun geöffnete Seite zeigt die Einstellungen für den Pi. Hier die gewünschte IP-Adresse eintragen und statisch machen.

![fritzbox_raspberrypi_settings](https://user-images.githubusercontent.com/123265893/218282458-1723be36-b3dd-47dd-bd05-67a59734c598.PNG) 

### _Pi-hole_ 

Jetzt installieren wir Pi-hole. Das sollte kein Problem sein. Man kann den folgenden Befehl verwenden: `curl -sSL https://install.pi-hole.net | bash`. "Piping to bash" ist ein umstrittenes Thema, da es den Benutzer daran hindert, den Code zu lesen, der ausgeführt werden soll. Wenn du den Code lieber vor der Installation überprüfen möchtest, bieten die Entwickler alternative Installationsmethoden an. Weitere Informationen sind [hier](https://docs.pi-hole.net/main/basic-install/) zu finden. 

Die Entwickler schlagen vor, den Router nach der Installation von Pi-hole zu konfigurieren und informieren über verschiedene Router-Modelle. Bezüglich der Fritzbox gibt es eine Anleitung in [Englisch](https://docs.pi-hole.net/routers/fritzbox/) und [Deutsch](https://docs.pi-hole.net/routers/fritzbox-de/). Für andere Router gibt es auch Anleitungen. Ich empfehle, die Änderungen im Auge zu behalten, da der Internetzugang möglicherweise vorübergehend blockiert ist und es nützlich sein kann, während des Konfigurationsprozesses eine Internetverbindung zu haben. 

Nach der Installation von Pi-hole und der Router-Konfiguration benötigt Pi-Hole noch Listen voller bösartiger Webadressen. Solche Listen sind z.B. [hier](/../../../../RPiList/specials/blob/master/Blocklisten.md) zu finden (hauptsächlich für deutsche Nutzer, daher sind die Anleitungen dort auch auf Deutsch). 

Melde dich im Pi-Hole an und klicke auf „ADLISTS“. Jetzt sollte diese Seite zu sehen sein: 

![pi-hole_adlists](https://user-images.githubusercontent.com/123265893/218336144-76b6f54d-b967-422d-bfd8-02afa6872aeb.png) 

Es können mehrere Listen auf einmal hinzugefügt werden, indem die Einträge mit Leerzeichen von einander getrennt werden. Mit anderen Worten: im oben genannten Repository können alle Listen auf einmal markiert werden, mit der rechten Maustaste anklicken und in die Zwischenablage kopieren. Dann in das Adressfeld in Pi-Hole einfügen. Nachdem die Listen hinzugefügt wurden `pihole -g` ausführen, um die Gravity-Datenbank zu aktualisieren. Dies kann auch innerhalb der Pi-hole-Oberfläche geschehen, aber dann muss diese Seite offen gehalten werden, bis das Update abgeschlossen ist. Ich empfehle die Kommandozeile. Die Aktualisierung der Datenbank dauert eine Weile, da die Listen validiert werden müssen. Also Geduld.

Wenn du das Passwort für Pi-hole vergessen hast, das während des Installationsvorgangs angezeigt wurde, einfach auf `Forgot Password` klicken. Es werden dann Anweisungen angezeigt, wie ein neues Passwort eingerichtet werden kann. Eine ssh-Verbindung wäre in diesem Moment sehr praktisch, falls kein Monitor mehr am Pi angeschlossen ist.

Übrigens wird während des Installationsvorgangs ein Cron-Job erstellt, um Pi-hole und die Gravity-Datenbank regelmäßig zu aktualisieren. Der Befehl `ls -l /etc/cron*` zeigt alle laufenden Cronjobs an. Die Ausgabe sieht so aus: 

![cronjob](https://user-images.githubusercontent.com/123265893/218337237-dd2cf85c-af31-453b-a2a0-fa37ecd07b2e.png) 

Um sicherzustellen, dass Pi-hole funktioniert, muss der Router und Pi-hole [konfiguriert](https://docs.pi-hole.net/main/post-install/) werden. Es empfiehlt sich, eine statische IP den Raspberry Pi einzurichten, da sonst Probleme auftreten können - spätestens wenn Unbound als lokaler DNS ausgeführt wird. Aber das ist bereits erledigt und sollte kein Problem mehr darstellen.

Außerdem habe ich Pi-hole **NICHT** als DHCP-Server eingerichtet. Das überlasse ich der Fritzbox. Angenommen, der Raspberry Pi funktioniert nicht und Pi-hole ist der DHCP-Server, dann ist jedes Gerät im Netzwerk blind, taub und stumm und dann muss wahrscheinlich der Router zurückgesetzt werden. Das wäre wirklich ärgerlich.

Die DNS-Einstellungen im Pi-hole sollten vorerst so aussehen:

![pihole-settings-dns](https://user-images.githubusercontent.com/123265893/218263468-af768d92-d14d-4c1d-970b-97ae23dad615.png)

Natürlich kann jeder andere verfügbare DNS ausgewählt werden.

Jetzt sollte Pi-hole betriebsbereit sein und du kannst dir eine weitere Tasse Kaffee oder Tee holen, da wir erst zur Hälfte fertig sind.

### _Unbound_

Um Unbound zur Zusammenarbeit zu bewegen, sind unter Debian Bullseye einige Schritte mehr nötig, als uns diverse Anleitungen im Internet versprechen. Es reicht nicht aus, es einfach zu installieren, aber es ist offensichtlich notwendig. In unserem Fall verwenden wir einfach die Version, die in den Debian-Standard-Repositories zu finden ist. Dies ist derzeit die Version 1.9.xx. Auf GitHub gibt es eine [neuere Version](/../../../../NLnetLabs/unbound)), aber die Verwendung des Pakets aus dem Standard-Repo vermeidet Kompilierungs- und Aktualisierungsprobleme.

| Wo wir gerade dabei sind: Ich kann nicht sagen, ob die hier gezeigten Schritte für Versionen >1.9.xx notwendig sind oder ob eine Neukonfiguration sogar zwingend erforderlich ist. Gerüchten zufolge benötigen neuere Versionen von Unbound bestimmte hier beschriebene Änderungen nicht mehr. Sie können dann überschrieben werden und alles läuft wie am Schnürchen - oder auch nicht. Die Zeit wird es zeigen. |
|:-|

Hier ist eine Übersicht über die Dateien, die in irgendeiner Weise eine Rolle spielen werden, und die Verzeichnisse, in denen sie zu finden sind:

```
+-etc (directory)
   |   dhcpcd.conf (file)
   |   resolv.conf (file)
   |   resolvconf.conf (file)
   |   
   +---dnsmasq.d (subdirectory)
   |       99-edns.conf (file)
   |       
   \---unbound (subdirectory)
       \---unbound.conf.d (subdirectory)
               pi-hole.conf (file)
               resolvconf_resolvers.conf (file)
               root-auto-trust-anchor-file.conf (file)
```

Wie zu sehen ist, findet das Ganze im Verzeichnis /etc/ oder in Unterverzeichnissen von /etc/ statt. Daher sind für jeden Schreibvorgang root-Rechte erforderlich. Ich werde das nicht jedes Mal erwähnen. 

Im Folgenden gehen wir Schritt für Schritt jede einzelne Datei durch und am Ende sollte Unbound gut mit Pi-hole funktionieren und die DNS-Anfragen entsprechend beantworten. 

#### /etc/dhcpcd.conf 

An dieser Datei sind wohl nur ein bis zwei Änderungen nötig, da während der Installation von Pi-hole die meisten benötigten Einträge geschrieben werden. Fast am Ende dieser Datei ist die folgende Zeile zu finden:

``` 
# fallback to static profile on eth0 
``` 
Unter dieser Zeile sollten die IP-Adressen des Raspberry Pi, des Routers und des DNS stehen und es wird wohl so ähnlich wie hier aussehen (natürlich ist es nicht wirklich eine Tabelle und es gibt keine Beschreibung):

| Eintrag | Beschreibung |
|:-|:-|
| #schnittstelle eth0 | diese Zeile nicht ändern |
| #fallback static_eth0 | diese Zeile nicht ändern |
| interface eth0 | Schnittstellenname, „eth“ zeigt eine Kabelverbindung mit einer Ethernet-Karte an, wobei die Zahl am Ende die Nummer der Schnittstelle angibt, manche Netzwerkkarten haben 2 oder mehr Anschlüsse; es sollte keine WLAN-Verbindung verwendet werden, da dies als instabil gilt|
| static ip_address=192.168.xxx.xxx/24 | <--- das muss die IP-Adresse Ihres Raspberry Pi sein, unter der der kleine Racker im Heimnetz erreichbar ist |
| static router=192.168.xxx.xxx | <--- dies muss die IP-Adresse Ihres Routers sein, Standard ist 192.168.178.1 |
| static domain_name_servers=127.0.0.1#5335 | <--- dies ist die IP-Adresse des DNS (es muss die localhost-Adresse sein); die Portadresse hinter der IP ist wichtig und muss angegeben werden, falls nicht vorhanden |

Die statische IP-Adresse und die Adresse des Routers hängen von dem im Router festgelegten Adressraum des Heimnetzwerks ab. Diese Einstellungen müssen mit den Einstellungen in Ihrem Router übereinstimmen. Der Adressraum zwischen 192.168.0.1 und 192.168.255.254 wird normalerweise für Heimnetzwerke und nicht für öffentliche Netzwerke verwendet. Auf diese Weise „weiß“ jedes Gerät und jede Anwendung, ob es/sie sich im Heimnetz befindet oder nicht. Die Fritzbox verwendet dies als Standardeinstellung, daher wird es bei dir ähnlich aussehen und nur die IP-Adresse für den Raspberry Pi muss geändert werden, wenn sie nicht korrekt ist. Die Portadresse `#5335` nach der localhost-Adresse 127.0.0.1 muss wahrscheinlich noch eingefügt werden. Jetzt sind wir mit dieser Datei fertig.

Wenn die Änderungen vorgenommen wurden, den DHCP-Server nicht neu starten, da dies zu einem (temporären) Verlust der Internetverbindung führen kann. Wir werden später einen Neustart durchführen, wenn alles erledigt ist.

#### _/etc/resolv.conf_

Diese Datei enthält nur zwei Zeilen:

```
# Generated by resolvconf
Nameserver 127.0.0.1
```

Die erste Zeile sagt uns, dass dies eine generierte Datei ist, daher bleiben Inhaltsänderungen nicht erhalten. Aber wir sehen, die Portadresse ist hier nicht angegeben. Daher muss sie (wie oben beschrieben) in der Datei dhcpcd.conf angegeben werden, damit sie in Verbindung mit der IP-Adresse 127.0.0.1 im Netzwerk angekündigt wird. Unbound erwartet die Anfragen über diesen Port, sonst funktioniert der DNS nicht.

Zur Erinnerung: im Moment ist die Fritzbox noch unser DNS und sie fragt einen öffentlichen DNS (hier DNS.Watch), wenn sie die angeforderte Adresse selbst nicht kennt. Aber das wollen wir ändern, also weiter im Text.

#### _/etc/resolvconf.conf_

Neuere Debian-basierte OS-Releases installieren automatisch ein Paket namens `openresolv`, das zu unerwartetem Verhalten von Pi-hole und Unbound führt. Openresolvs service/config weist resolvconf an, den eigenen DNS-Dienst von Unbound auf dem Nameserver 127.0.0.1 in die Datei `/etc/resolv.conf` zuschreiben, aber ohne den Port #5335 (siehe vorheriger Abschnitt). Die Datei `/etc/resolv.conf` wird von lokalen Diensten/Prozessen verwendet, um die konfigurierten DNS-Server zu bestimmen. Openresolv muss entfernt und der Dienst deaktiviert werden. Oder die Konfigurationsdatei muss geändert werden, um die Fehlkonfiguration zu umgehen.

Es gibt zwei Möglichkeiten, das Problem zu lösen, und manche werden

##### _den schweren Weg_

wählen nur für den Fall, dass `unbound-resolvconf.service` eines Tages benötigt wird. In diesem Fall muss `/etc/resolvconf.conf` modifiziert werden. Ganz am Ende der Datei ist die folgende Zeile zu finden:

`unbound_conf=/etc/unbound/unbound.conf.d/resolvconf_resolvers.conf`

Diese Zeile muss auskommentiert werden:

`#unbound_conf=/etc/unbound/unbound .conf.d/resolvconf_resolvers.conf`

Dieser Schritt ist notwendig, da resolvconf eine unerwünschte Datei erstellt, wenn der Eintrag aktiv ist. Nachdem der Eintrag auskommentiert wurde, sollte die unerwünschte Datei umbenannt werden mit

`mv /etc/unbound/unbound.conf.d/resolvconf_resolvers.conf resolvconf_resolvers.conf.backup`

oder mit

`rm /etc/unbound/unbound.conf.d/resolvconf_resolvers .conf`

gelöscht werden.

Wird einfach die Datei entfernt und Unbound neu gestartet, funktioniert Unbound korrekt als rekursives DNS. Das Problem ist, dass die o.g. Datei nach einer Weile wieder automatisch generiert wird, was in Buster nicht passiert ist, aber jetzt in Bullseye wegen des `openresolv`-Pakets. Um dies zu verhindern, muss die Datei `/etc/resolvconf.conf` wie oben beschrieben modifiziert werden, oder du wählst

##### _den leichten Weg_

Prüfe mit `$systemctl status unbound-resolvconf.service `, ob unbound-resolvconf.service aktiv ist.

Wenn der Service läuft (wahrscheinlich wird er das unter Bullseye), mit den folgenden Befehlen deaktivieren, um den Start bei einem Reboot verhindern:

```
$systemctl disable unbound-resolvconf.service
$systemctl stop unbound-resolvconf.service
```

Dann die Datei löschen:

`/etc/unbound/unbound.conf.d/resolvconf_resolvers.conf`

oder mit

`rm /etc/unbound/unbound.conf.d/resolvconf_resolvers.conf`

umbenennen.

Weitere Informationen sind [hier](https://docs.pi-hole.net/guides/dns/unbound/#disable-resolvconf-entry-for-unbound-required-for-debian-bullsye-releases) zu finden.

#### _/etc/dnsmasq.d/99-edns.conf_

Die Entwickler schlagen vor, das Hinzufügen von `edns-packet-max=1232` zu einer Konfigurationsdatei wie `/etc/dnsmasq.d/99-edns.conf` in Betracht zu ziehen, um FTL zu signalisieren, das Limit einzuhalten.

1) `$touch /etc/dnsmasq.d/99-edns.conf`
2) `$nano /etc/dnsmasq.d/99-edns.conf`
3) `edns-packet-max=1232` einfügen, dann `Strg-o`, `Enter` und `Strg-x` (in dieser Reihenfolge)

oder Midnight Commander verwenden, um diese Operationen auszuführen.

#### _/etc/unbound/unbound.conf.d/pi-hole.conf_
 
Diese Datei muss eventuell noch angelegt werden (`touch /etc/unbound/unbound.conf.d/pi-hole.conf`) und enthält Einstellungen, die [hier](https://docs.pi-hole.net/guides/dns/unbound/#configure-unbound) zu finden sind. Grundsätzlich kann der Inhalt so übernommen werden. Die einzige(!) Änderung, die ich vorgenommen habe, ist `do-ip6: no` zu `do-ip6: yes`. Aus irgendeinem Grund scheint Unbound (Version 1.9.xx) IP6 verwenden zu wollen. Nach langem Suchen habe ich in den Tiefen des Internets einen Hinweis gefunden, dass dies der Grund sein könnte, warum Unbound nicht mitspielt. Nachdem ich diesen Eintrag geändert hatte, funktionierte es. Das kann daran liegen, dass dies auch in der Fritzbox standardmäßig aktiviert ist und ich es nicht abgeschaltet habe. Später wird es diesbezüglich jedoch seltsam, wenn es darum geht, Pi-hole zu sagen, dass es Unbound als DNS verwenden soll.

#### _/etc/unbound/unbound.conf.d/resolvconf_resolvers.conf(.backup)_

Dies ist der Übeltäter. Die Datei löschen oder umbenennen und den `unbound-resolvconf.service` deaktivieren oder zumindest `/etc/resolvconf.conf` [manipulieren](/../../../../TomfromBerlin/Debian-Pihole-Unbound/blob/main/README_de.md#den-schweren-Weg), um die Wiederbelebung der Datei zu verhindern.

#### _/etc/unbound/unbound.conf.d/root-auto-trust-anchor-file.conf_

Standard ist /usr/share/dns/root.key. Wenn die Datei nicht existiert oder leer ist, wird ein eingebauter Schlüssel hineingeschrieben. Wenn Unbound aus dem Debian-Repository installiert wurde, wurde das Paket unbound-anchor als Abhängigkeit installiert, sodass diese Datei bereits vorhanden sein sollte und sie nicht bearbeitet werden muss.

Außerdem muss die Datei root.hints nicht heruntergeladen werden, wenn Sie Unbound aus den Debian-Repositories installiert haben, da sie automatisch installiert und aktualisiert wird. Unbound findet die Datei dann automatisch und verwendet sie, um die Domänenauflösung zu bootstrappen.

## _Pi-hole und Unbound_

Jetzt können wir Pi-hole mitteilen, dass wir einen lokalen DNS haben. Dazu z.B. von der Workstation aus Pi-Hole im Webbrowser unter der folgenden Adresse öffnen:

`http://IP-Adresse-des-Raspberrypi/admin/` (nicht die localhost-Adresse verwenden, sondern die im Router angegebene)

Nach der Anmeldung unter „Einstellungen“ zur DNS-Seite navigieren:

![pihole-settings-local_dns](https://user-images.githubusercontent.com/123265893/218285445-b98d1e23-0ced-4c18-9821-2099dfd94d3a.png)

`127.0.0.1#5335` in das Feld Custom1 (IPv4) eingeben, speichern(!) und den Raspberry Pi neu starten (`sudo shutdown --reboot`).

Aber noch etwas muss getan werden: Der Router muss wissen, dass er nicht mehr für die DNS-Auflösung zuständig ist. Dies wird erreicht, indem die IPv4-Adresse des lokalen DNS im Router auf die IP-Adresse des Raspberry Pi gesetzt wird. Dazu im Router-Interface zu folgender Seite navigieren und unten rechts auf die Schaltfläche IPv4-Einstellungen klicken.

![Heimnetz-netzwerk-ip4-ip6_einstellungen-button](https://user-images.githubusercontent.com/123265893/218287228-8c4b4638-ab45-4ca6-ba06-bd89c64b04e1.png)

Im nächsten Bildschirm können Sie die IP eingeben Adresse:

![Heimnetz-netzwerk-ip4_einstellungen](https://user-images.githubusercontent.com/123265893/218287360-216c02ec-75c8-48c7-ac79-feeecf985de3.PNG)

Alle anderen Felder unberührt lassen.

Sollte nach dem Neustart kein Internetzugang zur Verfügung stehen, kann auf der [DNS-Einstellungsseite](/../../.././TomfromBerlin/Debian-Pihole-Unbound/blob/main/README_de.md#pi-hole-und-unbound) von Pi-hole im Feld `Custom IPv6` die Adresse `::1` eingegeben werden. Das hat bei mir funktioniert. Vielleicht hast Du auch den kleinen gelben Punkt im oberen linken Bereich bemerkt. Dieser signalisiert, dass möglicherweise ein Problem vorliegt. Ein Klick darauf öffnet folgende Seite:

![pihole-warning](https://user-images.githubusercontent.com/123265893/218285573-443eb6fa-7344-4600-a8a0-a51d4907f54a.png)

Dies sagt uns, die IP6-Adresse sei irgendwie redundant, da es sich um die Adresse von localhost handelt, die bereits durch die IPv4-Adresse (127.0.0.1) definiert ist. Diese wird jetzt von Pi-Hole ignoriert und wir könn(t)en das auch. Jetzt sollten wir von jedem Gerät in unserem Heimnetzwerk Internetzugang haben und jede Anfrage sollte von Unbound beantwortet und von Pi-Hole gefiltert werden.

Aber da ist dieser kleine, winzige, gelbe Punkt oben links ... Und hier kommt das Seltsame ins Spiel: Die IP6-Adresse `::1` kann jetzt gelöscht werden und alles funktioniert immer noch. Sehr seltsam!

Die erste Suche nach einer Website suchen kann es eine Weile dauern, bis sie gefunden wird. Der zweite und jeder weitere Anruf ist im Handumdrehen bearbeitet.

Erste Suche mit `dig pi-hole.net 127.0.0.1 -p 5335` (achte auf die Abfragezeit ('Query time', vierte Zeile von unten)):

![first-dig-noerror-107ms](https://user-images.githubusercontent.com/123265893/218286774-15835d42-a999-4db4-8456-d7940ba678b8.png)

Zweite Suche:

![second-dig-noerror-0ms](https://user-images.githubusercontent.com/123265893/218286806-2a52dcf4-bc63-40f4-8d93-c0e6c5ecce6e.png)

Die vierte Zeile von oben sollte `status: NOERROR` lauten. Wenn ein Fehler auftritt, dann stimmt offensichtlich etwas nicht. Überprüfen Sie alle Schritte, die unternommen wurden. Vielleicht ist da ein Tippfehler oder so. Sollte sich im Tutorial ein Fehler eingeschlichen haben, wäre ich für Hinweise darauf sehr dankbar. Geeignete Wege sind über die [Diskussionsseite](/../../../../TomfromBerlin/Debian-Pihole-Unbound/discussions) oder über eine [Problemmeldung](/../../../. ./TomfromBerlin/Debian-Pihole-Unbound/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc).

Das war's, Leute. Ich hoffe, das alles hilft etwas.
