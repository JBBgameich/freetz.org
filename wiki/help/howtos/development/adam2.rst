.. _ADAM2-Bootloader:

ADAM2-Bootloader
================

ADAM2 ist ein Bootloader von Texas Intruments, der ähnliche Aufgaben
übernimmt wie ein BIOS beim PC. ADAM2 wurde in abgewandelter Form von
AVM in frühen FRITZBox-Modellen mit Kernel 2.4 eingesetzt, und wich noch
vor Einführung des Kernels 2.6 der aus Nutzersicht nahezu
funktionsgleichen Eigenentwicklung EVA. Wegen der vielen Parallelen
können beide Varianten als "der Bootloader" betrachtet werden, und
müssen nur in Einzelfällen unterschieden werden.

Die Aufgaben des Bootloaders sind:

-  die Hardware zu initialisieren
-  das Flash zu erkennen
-  das RAM zu erkennen und zu prüfen
-  die Partitionierung und grundlegende Werkseinstellungen zu verwalten
-  eine Serielle Konsole bereitzustellen
-  einen kleinen FTP-Server bereitzustellen (für Recovery)
-  das Environment im TFFS zu verwalten
-  das installierte Kernel zu booten

.. _Bootloader-Backupanlegen:

Bootloader-Backup anlegen
-------------------------

Wer will kann sich ein Bootloader-Backup anlegen sollte sich aber
dringend merken von \*genau welcher\* Box das war (MAC-Adresse). Mehr
dazu um nächsten Abschnitt.

Die Grösse des Bootloaders wird in der Environment-Variable ``mtd2``
gespeichert, die fest im Bootloader selbst eingestellt ist.

Aus Linux heraus hat diese Partition oft eine von ``2`` abweichende
Nummer die man mit folgendem Befehl ausfindig macht:

.. code:: bash

   cat /proc/mtd

Eine der dort genannten Partitionen nennt sich ``bootloader`` oder
``urlader``. Mit deren Nummer (hier z.B. ``3``) liesst man dann die
zugehörige ``mtdblock`` Device aus:

.. code:: bash

   cat /dev/mtdblock3 > bootloader.bin

Siehe auch `Flash-Partitionen im laufenden Betrieb
sichern <save_mtd_1.html>`__.

ADAM2 ist immer 64KB gross, EVA bei älteren Modellen 64KB, bei neueren
Modellen 128KB oder 256KB. Beim IAD 7570 ist die ``mtd2`` Partition zwar
256KB, die oberen 128KB sind jedoch leer (0xFF). Dies könnte auf eine
geplante (aber technisch nicht durchführbare) zweite Instanz des
Bootloaders deuten. Die Grösse des Bootloader sollte unbedingt beim
Entwickeln von Aliens beachtet werden. Ein nicht angepasstes ``install``
Script für 64KB Bootloader zerstört einen 128KB Bootloader ohne
Vorwarnung! Das gilt dann auch für das AVM Webinteface - die Box wird
vorhersagbar zum Brick.

Daher, wer Aliens entwickelt ohne das Environment des Zielgerätes
geprüft und mit dem ``install`` Script verglichen zu haben gefährdet
Geräte. Auch im Trunk sollten daher solche riskanten Experimente nur mit
Aufwand freischaltbar sein (implementiert: neuer nicht per GUI
aktivierbarer "Real Developer" Risiko-Modus).

.. _Bootloaderüberschreiben:

Bootloader überschreiben
------------------------

Kurz: **NEIN!**

Der Bootloader enthält zahlreiche Informationen die eine Box einmalig
machen, bei vielen WLAN-Modellen auch Kalibrierung ohne die das Gerät
nicht mehr das selbe ist.

Eine Übertragung ist daher grober Unfug. Auch wenn man diese Angaben
korrekt anpassen würde gibt es ein noch fataleres Problem:

Selbst gleiche Modelle wurden je nach Verfügbarkeit mit
unterschiedlichen Flash- und RAM-Chips bestückt, besonders bei RAM auch
mit Bausteinen mit erheblich abweichenden Eigenschaften (wie Anzahl der
Banks und Timing etc.). Diese Unterschiede werden durch undokumentierte
Konfiguration im Bootloader gesichert. Bootloader-Updates von AVM
übertragen diese Information.

**Ein Bootloader ist auch zwischen gleichen Modellen nicht gefahrlos
übertragbar!.**

Ohne RAM funktioniert auch ein intakter Bootloader nicht ⇒ Brick.

Um die oben erstellte Sicherung \*auf genau die selbe Box\*
zurückzuspielen \*wäre\* dies der Weg:

.. code:: bash

   cat bootloader.bin > /dev/mtdblock3

Selbst dann besteht Brickgefahr. Die MTD Treiber blockieren nicht das
OS. Greift ein anderer Prozess währenddessen z.B. über die ADAM2 API auf
das Environment zu hängt sich das System eventuell während des Schreib-
oder Löschvorgangs auf ⇒ Brick.

Grundsätzlich sollte man daher den Bootloader nur mit geeigneter AVM
Firmware schreiben oder wenn man über Werkzeuge zum debricken (EJTAG)
verfügt.

.. _Bootloader-Befehle:

Bootloader-Befehle
------------------

-  Die über die Serielle Konsole nutzbaren Befehle findet man im `​ADAM2
   Shell <http://www.wehavemorefun.de/fritzbox/ADAM2_Shell>`__ Artikel
-  Die per FTP nutzbaren Befehle findet man im
   `​TinyFTP <http://www.wehavemorefun.de/fritzbox/TinyFTP>`__ Artikel

Bei einigen Modellen wurde aus Sicherheitsgründen die ADAM2 Shell
entfernt. Dies betrifft keine im freien Handel befindliches Geräte, nur
Providermodelle wie die FRITZBox Cable.

Per FTP sind nur Modelle mit mindestens einem LAN-Port erreichbar (und
recoverbar). Daher eignen sich Modelle ohne LAN (einige Repeater) nicht
zum Experimentieren oder Freetzen. Grobe Faustregel: Wenn AVM eine
Recovery bereitstellt ist ein Gerät perfekt zum Freetzen geeignet. Dies
gilt nicht automatisch für von kleineren Providern bereitgestellte
Geräte! Diese können sogenannte "Provider Additive" enthalten die einen
Werksreset überstehen. Neuere Recoveries verweigern bei solchen Geräten
ihre Funktion, ältere Recoveries zerstören das Additiv (ohne "vor Ort"
Hilfe des Providers irreparabel). Dies dürfte der Grund sein warum AVM
die `​7570
Recovery <ftp://ftp.avm.de/fritz.box/fritzbox.fon_wlan_7570/x_misc/english/>`__
vom FTP-Server entfernte.

Für AVM Speedports gab es nur werksinterne Recoveries für den
Telekom-Service. Diese wurden leider nie veröffentlicht.

**Achtung: Im Netz kursieren auch defekte Speedport (sp2fr) Recoveries
die jeden Speedport bricken!**

Speedports lassen sich mit geringem Aufwand auch sauber mit Freetz
recovern. Achtung: Howtos, Forenpostings und Windows Tools die MTD3/4
clean empfehlen sind entweder uralt oder ein stümperhafter
Faulheitshack! Details zu den teilweise fatalen Folgen dieser fossilen
Unsitte folgen.

.. _Bootloader-Quelltext:

Bootloader-Quelltext
--------------------

ADAM2 wurde vielen Abnehmern von TI-Chips bereitgestellt und war
eigentlich nie quelloffen. Jeder Hersteller von Geräten modifizierte ihn
dann nach eigenen Bedürfnissen und hielt den Quelltext geschlossen, so
auch AVM. Auch Linksys nutzte eine modifizierte ADAM2 Version, leakte
den Quelltext aber versehentlich in einem wag54g Tarball. Damit änderte
sich nicht der proprietäre Status von ADAM2, er wurde aber zumindest in
der Linksys Variante "Visible Source" und kann
`​hier <http://gpl.back2roots.org/source/WAG54GV2/src/Adam2/>`__
gestöbert werden. Diese Variante ist aber nur sehr beschränkt für die
FRITZBox aussagekräftig.

Der Quelltext der AVM Variante von ADAM2 wurde nie veröffentlicht.
Lediglich die `​ADAM2
API <http://gpl.back2roots.org/source/fritzbox/ALL_4.06/GPL-release_kernel/linux/drivers/adam2/>`__
zum Erreichen des Environments war quelloffen.

Der Nachfolger EVA basiert nicht auf ADAM2 und ist ein kompletter
funktionskompatibler Rewrite. Im Gegensatz zu ADAM2 unterstützt EVA
direkt komprimierte Kernels und wurde bisher auf mindestens 8
Architekturen portiert. ADAM2 kam nur auf AR7-Modellen mit Kernel 2.4
zum Einsatz. Alle von Freetz erzeugte Firmware benötigt Kernel 2.6 und
EVA.

.. _AufbaudesBootloaders:

Aufbau des Bootloaders
----------------------

Am Anfang eines jeden MIPS-Bootloaders befindet sich eine 8-Byte
"Signatur". In Wirklichkeit handelt es sich um Assembler-Code zur
Initialisierung des MIPS-Kerns die MIPS netterweise bittet nicht zu
ändern. Diese Befehlssequenz löscht 2 Hälften eines Debug-Registers
(Watchpoint Exception bei "Berührung" einer Adresse) die im
Normalbetrieb nicht genutzt werden und eignen sich auch wegen der Länge
hervorragend als zuverlässige Signatur. Siehe in diesem
`​Quelltext <http://gpl.back2roots.org/source/WAG54GV2/src/Adam2/src/avreset.S>`__
den Kommentar "First thing: clear watch regs".

Für Litte Endian Modelle (AR7, UR8) assembliert dies zur Hexfolge
``00 90 80 40 00 98 80 40`` die immer am Anfang von ``mtd2`` (also vom
gesamten Flash) zu finden ist. Bei Big Endian Modellen (AR9, AR10, VR9,
Fusiv) entspricht es der 32-bit gespiegelten Hexfolge
``40 80 90 00 40 80 98 00`` und es befinden sich grundsätzlich weitere
Daten davor. Dies ist eine bis zu 1024 Bytes grosse
`​Vektortabelle <https://dev.openwrt.org/browser/trunk/package/uboot-ifxmips/files/cpu/mips/danube/start.S>`__
oder Kalt- und Warmstartvektoren und Code zur Initialisierung der `​hier
ab Zeile
44 <http://code.metager.de/source/xref/denx/u-boot/arch/mips/cpu/mips32/start.S#44>`__
genannten EBU-Einheit. Beim AR9, AR10 und VR9 sind dies 24 Bytes (Offset
0x18), beim Fusiv die vollen 1024 Bytes (Offset 0x400). Diese Bytes
gehören natürlich zum Bootloader, die beiden "Signatur-Befehle"
verschieben sich dadurch lediglich.

Zum Ausmaskieren von ARM Bootloadern sind diese Signaturen nicht
geeignet, da ARM Assembler andere Häufigkeitsverteilungen hat. Ein
zuverlässiger Detektor muss also zuerst ARM Code erkennen. Zum Erkennen
von Puma5 (ARM1176BE) Bootloadern gibt es auch eine zuverlässige
Assemblersequenz aus der Lowlevel-Initialisierung. Siehe in
`​diesem <http://gpl.back2roots.org/source/puma5/netgear/CMD31T_GPL/ti/psp_uboot/src/u-boot-1.2.0/cpu/arm1176/puma5/puma5.h>`__
und
`​diesem <http://gpl.back2roots.org/source/puma5/netgear/CMD31T_GPL/ti/psp_uboot/src/u-boot-1.2.0/board/tnetc550/lowlevel_init.S>`__
Quelltext den Kommentar "Unlock CFG MMR region". Dies assembliert zu
Code der die Hexfolge ``08 61 1A 38 83 E7 0B 13 08 61 1A 3C`` enthält.
Diese Signatur ist leider nicht am Anfang des Bootloaders zu finden. Bei
der 6360 mit EVA 2070 befindet sie sich an Offset 0xF1AC, also noch in
den ersten 64KB von Puma5 EVA. Leider stehen keine Recoveries zum Testen
der Signatur zur Verfügung. Mit Puma5 EVA oder `​U-Boot
Code <http://gpl.back2roots.org/source/puma5/netgear/CMD31T_GPL/ti/psp_uboot/src/u-boot-1.2.0/board/tnetc550/lowlevel_init.o>`__
funktioniert sie einwandfrei.

Ungeachtet des Offsets erkennt man EVA am 32-bit Wert ``0x00000002``
oder ``0x00000003`` im jeweiligen Endian an Offset 0x580. Dies ist die
Version (fast immer 2, bei brandaktuellen Modellen auch 3) der EVA
`​Urlader-Konfig <http://www.wehavemorefun.de/fritzbox/ADAM2#Urlader-Konfig>`__
in der Teile der Grundeinstellungen im Werk eingetragen werden. Da
EVA-Images in Firmware keine Konfiguration enthalten ist der Wert dort
``0xFFFFFFFF``. Auch ADAM2 enthielt Teile dieser Einstellungen, jedoch
einkompiliert ohne festen Offset.

In beiden Bootloadern sind 8 Default-MAC-Adressen ``00:04:0E:FF:FF:01``
- ``00:04:0E:FF:FF:08`` einkompiliert, die Mindestanforderung für eine
Kommunikation, sollte die Urlader-Konfig defekt oder noch nicht
vorhanden sein. Seit Entwicklung des `​VoIP Gateway
5188 <http://www.wehavemorefun.de/fritzbox/5188>`__ findet man in EVA
auch das Environment der zweiten CPU fest einkompiliert, da diese über
kein eigenes Flash und daher auch kein TFFS und Environment verfügt und
über NFSRoot bootet. Environment-Variablen können intern nicht nur per
Name angesprochen werden sondern auch per numerischem Index. Dazu wurde
eine Liste numerisch ansprechbarer Variablen einkompiliert die immer mit
``AutoMDIX`` anfängt. Bei älteren ADAM2 Urladern endet die Liste nach
der letzten Variable (z.B. ``wlan_key``), bei neueren ADAM2 und EVA mit
``zuende``. Diese Tabelle ist quelloffen, da sie auch die Namenstabelle
des TFFS ist, siehe "#if defined(URLADER)" und "_TFFS_Name_Table" in
`​tffs.h <http://gpl.back2roots.org/source/fritzbox/7270_5.05/GPL-release_kernel/linux/include/linux/tffs.h>`__.

In allen Recoveries finden sich Fragmente von mindestens einem
Bootloader. In den Anfängen der FRITZBox wurde identische Firmware für
mehrere Modelle umbenannt, die Bootloader jedes dieser Modelle waren
jedoch noch nicht harmonisiert. Entsprechend findet man in Recoveries
aus dieser Zeit multiple Bootloader-Signaturen, da der modellspezifische
Teil mehrfach enthalten war, die modellübergreifende Teil jedoch nicht.

Grundsätzlich ist die Extraktion eines funktionierenden Bootloaders aus
einer Recovery nicht möglich, da der Bootloader aus in der Recovery
enthaltenen Codefragmenten und auf der Box befindlichen
Werkseinstellungen intelligent zusammengebaut wird. Für die
Modellforschung ist die Auffindbarkeit der Fragmente und deren
Grundeinstellungen jedoch interessant. Von 436 analysierten Recoveries
waren etwa 14% ADAM2-MIPSLE, 50% EVA-MIPSLE und die restlichen 36%
EVA-MIPSBE. Bei allen Proben genügte die Auswertung der letzten 256 KB
des mit 7zip isolierbaren ``.data`` Segments jeder Recovery.exe.

Bei der Umstellung auf Kernel 2.6 mussten einige Modelle auf EVA
umgestellt werden. Daher enthalten einige Firmware-Updates ein
``urlader.image`` und passende Programme zur Aktualisierung. In den
Anfängen gab es auch einige ADAM2-Updates, in denen der Dateiname
Modell- und Versionsinformation wie
``urlader.Fritz_Box_4MB.97.adam2.image`` enthielt. Im Gegensatz zu den
Fragmenten in Recoveries sind diese immer "fixed-size" Bootloader mit
Leerbereichen für zu übertragende Konfiguration.

In ADAM2 ist die Version des Urladers als Integer in der Form
``urlader-version \x00 99 \x00`` einkompiliert, bevor es diese Variable
gab in der Form ``$ProjectRevision: 1.24 $``, auch mit mehrstelliger
Version wie in
`​diesem <http://www.akk.org/~enrik/fbox/OLD/boot-06.01.116.txt>`__
Bootlog. Bei EVA findet man die Version mit bis zu 3 Bytes Abstand vor
oder hinter der Zeichenfolge ``%d.%s`` und es muss 1000 hinzugezählt
werden. Ein zusätzliches ``M`` signalisiert eine modifizierte Variante.
In neueren Recoveries findet man zudem einen ``.eva`` Dateinamen, z.B.
für die 7360v2 mit EVA 2717M den String ``1717M.eva``. In 436
analysierten Recoveries wurden ADAM2 der Versionen 1.20, 1.24, 50 bis 99
und EVA der Versionen 1124 bis 2970 entdeckt (Stand 2014-01). Bei 5 EVA
1190 Recoveries 04.30/31 funktioniert die Versionserkennung nicht, die
Nummer ist dort irgendwo ab Offset 0xF000 relativ zur Signatur zu
finden. Diese müssen per MD5 erkannt werden.

Der älteste in Firmware gefundene Bootloader, ADAM2 Version 1.24, wurde
in der bisher ältesten bekannten Firmware fritz.box_sl.05.01.63.image
vom 30. April 2004 (1 Monat nach Vorstellung der ersten FRITZBox auf der
CeBIT) entdeckt. 4 englische Bootloader der Version 1.20 sind neuer.
Durch getrennte Weiterentwicklung je Modell können ADAM2 Versionen nicht
kalendarisch sortiert werden. Der älteste in Firmware gefundene EVA
Bootloader Version 1124 befindet sich in einer frühen 7170 Recovery.
EVA-Versionen lassen sich auch erst ab etwa 1600 modellübergreifend
kalendarisch sortieren.

Recoveries enthalten zwei weitere leicht zu findende Versionsangaben für
den Programmteil. Über die enthaltenen Firmwarekomponenten sagen sie
nichts aus. Je 2 Beispiele:

-  FW 3.37:
   ``AVM Berlin recover-tool-version:[RECOVER:53][IO_CSP:11] compiled at Feb 18 2005 on 14:24:36``
-  FW 6.01:
   ``AVM Berlin recover-tool-version:[RECOVER:378M][IO_CSP:248] compiled at Aug 23 2013 on 13:52:14``
-  FW 3.37:
   ``[AVM Berlin Wizard Base Project, $ProjectRevision: 1.7 $, $Date: 2005/02/11 10:47:18Z $, kompiliert am Feb 14 2005 um 10:10:13]``
-  FW 6.01:
   ``[AVM Berlin Wizard Base Project, $ProjectRevision: 1.63 $, $Date: 2011/07/04 11:49:20Z $, kompiliert am Jul  8 2013 um 11:45:45]``

Wie man sieht werden die Komponenten GUI (Wizard), Recover- und I/O-Teil
getrennt entwickelt und kompiliert. Eine heutige Recovery besteht also
aus mindestens 6 Projekten. Auf älteren FRITZBox CDs (z.B. 3020)
befindet sich eine recover.exe ohne integrierte Firmware (etwa 100KB)
der noch ein externes Image bereitgestellt werden musste. Das Programm
nennt sich ``ar7recover`` und stammt vom Februar 2004, 1 Monat vor
Vorstellung der ersten FRITZBox. Dies dürfte wohl die älteste
veröffentlichte Recovery-Lösung von AVM sein.

Jede Recovery erkennt eine Box an der Urlader-Variable ``HWRevision``.
Da der Urlader keinen Zugriff auf den vollständigen Namen eines Modells
hat enthält jede Recovery eine Liste aller bis zum Erstellungsdatum
bekannten HWRevisions und deren Modellnamen. Die Liste befindet sich im
mit ``7zip`` isolierbaren ``.rdata`` Segment, bei älteren Recoveries bis
etwa 04.43 ist sie im ``.data`` Segment oder nicht vorhanden (bisher nur
bei einer 03.14). Zweck der Liste ist die menschenlesbare Anzeige des
gefundenen Modells in der GUI, unabhängig davon ob die Recovery passt.

Die Liste ist bis zu 3 KB gross und fängt immer mit dem String
``unknown`` an, gefolgt von nullterminierten Paaren HWR / Boxname, mit
32-bit Padding je String. Der letzte Eintrag ist immer die
``FRITZ!Box SL`` und ihre HWR ``F``. Einige Listen ordnen ``unknown``
die HWR ``K`` zu, bei anderen folgt direkt der erste Modellname. Die
Zuordnungpaare sind leider nicht konsistent. So enthält die Liste auch 2
aufeinanderfolgende Namen oder Nummern, aber auch Firmennamen wie AVM
und Telekom. Sie muss also intelligent interpretiert werden. Obwohl die
HWR-Liste auch in aktuellen Recoveries enhalten ist pflegt AVM sie seit
HWR 190 nicht weiter. Recoveries höherer Werte haben zusätzlich den
Zuordnungseintrag für das unterstützte Modell im ``.data`` Segment, die
HWR gefolgt von mehreren Nullbytes gefolgt vom Namen. Da bei allen
neueren Recoveries das ``.data`` Segment mit der nullterminierten HWR
anfängt kann diese als Suchstring in den letzten 256 KB des Segments
genutzt werden. Ob und wie bei diesen Recoveries Fremdmodelle mit HWR >
190 erkannt werden muss noch geprüft werden.

Bei der Analyse von 419 Recoveries mit obigem Wissen wurde `diese
Häufigkeitsverteilung </attachment/wiki/help/howtos/development/adam2/hwr.txt>`__\ `​ </raw-attachment/wiki/help/howtos/development/adam2/hwr.txt>`__
(tab delimited csv) ermittelt. Um Konsistenzfehler auszumaskieren
enthält die Liste nur Zuordnungen die in mindesten 5 Recoveries gefunden
wurden. Die Zähler für HWR > 190 wurden zuvor mit 10 multipliziert.

Die HWR am Anfang des ``.data`` Segments ist Teil einer Struktur, die in
jeder Recovery mit Kernel 2.6 zu finden ist. Bei neueren Recoveries
findet sie sich an Offset 0, bei älteren an Offset 64 (0x40). Sie
Struktur enthält die unterstützte HWR an Offset 0, die Sprache an Offset
16 (0x10) und die mit Leerzeichen getrennte Liste der unterstützten
Brandings an Offset 32 (0x20). Dies ist sehr nützlich da z.B. EWE
Recoveries nicht am Dateinamen erkennbar sind. Ein vierter String dessen
genauer Zweck noch unklar ist fängt normalerweise an Offset 0x30 an und
verschiebt sich um jeweils 8 Bytes wenn der Brandings-String länger als
8 Bytes ist. Bei Congstar (und vermutlich auch Telekom) Recoveries
enthält er ``tcom``, bei allen anderen enthält er unabhängig von Sprache
und Anbieter immer ``avm``.

In allen Kernel 2.4 Recoveries befindet sich am Ende des ``.data``
Segments eine Reihe von mit einem oder mehreren Nullbytes terminierten
Strings. Dies ist eine 2- oder 3-stellige Zahl unbekannten Zwecks
(leider nicht die HWR) oder der String ``IE``, der optionale String
``en`` oder ``de`` und die Firmwareversion in punktierter Schreibweise
(z.B. ``29.04.01``) mit dem optionalen Zusatz
``-prerelease-<checkpoint>``. Dahinter steht der optionale String
``avm`` oder ``freenet``, gefolgt von der optionalen Liste der
unterstützten Brandings. Lediglich die älteste bekannte Recovery mit
integrierter Firmware (03.14) enthält diese Information nicht. Sie ist
deutsch und kannte noch kein Branding. Die HWR muss bei Kernel 2.4
Recoveries aus dem Urlader ermittelt werden.

Der Vergleich der ermittelten Brandings mit den /etc Defaults bestätigt
die Zuverlässigkeit obiger Methoden, sowohl bei Release- als auch bei
Labor-Recoveries. Es existieren allerdings 2 Labor-Recoveries die einen
falschen Bootloader enthalten. Die Datei
``FRITZ.Box_2110.04.47-9457.recover-image.exe`` enthält einen HWR 130
Bootloader einer nie auf den Markt gekommenen 2121, die Datei
``fritz.box_fon_wlan_7050.04.50.B.telnet.recover-image.exe`` den HWR 94
Bootloader einer 7170.

.. _BootloaderundFreetz:

Bootloader und Freetz
---------------------

Da Freetz EVA benötigt sind einige Modelle schon vom Bootloader her
nicht für Freetz geeignet. Grundsätzlich sollte jede Box vor dem
Freetzen mit Originalfirmware aktualisiert werden. Dies aktualisiert
ggf. auch den Bootloader. Für einige ältere Modelle ist evtl. ein
`​Zwischenupdate <ftp://service.avm.de/Zwischenupdate/>`__ notwendig.

Für folgende Modelle existiert kein EVA Update:

-  FRITZBox (alle Versionen)
-  FRITZBox SL
-  FRITZBox 2030
-  FRITZBox Fon (Deutsch A/CH Annex A+B) - mit Tricks evtl. deutsch oder
   englisch aktualisierbar
-  FRITZBox Fon ata (alle Versionen)
-  FRITZBox Fon WLAN (Deutsch A/CH Annex A+B) - mit Tricks evtl. deutsch
   oder englisch aktualisierbar

Für einige dieser Modelle könnte Freetz ein EVA Update einer anderen Box
Alien patchen. Bei der FRITZBox SL und 2030 mit 2MB Flash und 8MB RAM
wird es wohl nie Freetz geben.

Anhänge (1)
~~~~~~~~~~~

-  `hwr.txt </attachment/wiki/help/howtos/development/adam2/hwr.txt>`__\ `​ </raw-attachment/wiki/help/howtos/development/adam2/hwr.txt>`__
   (4.0 KB) - hinzugefügt von *hippie2000* `vor 4
   Jahren </timeline?from=2014-01-19T21%3A15%3A59Z&precision=second>`__.
   “Komplette HWR-Liste aller 419 bekannten Recoveries, CSV”

Alle Anhänge herunterladen als:
`.zip </zip-attachment/wiki/help/howtos/development/adam2/>`__
