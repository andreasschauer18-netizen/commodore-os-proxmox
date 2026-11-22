<div align="center">

# 🕹️ Commodore OS Vision 3.1 auf Proxmox VE

### Retro-Desktop trifft Virtualisierung

**Eine erprobte Installation unter Proxmox VE 9.2.3 – inklusive Lösung für verschwindende Desktop-Elemente.**

![Gestaltete Projekt-Collage zu Commodore OS, Proxmox und Marco](assets/commodore-os-collage.png)

<sub>Gestaltete Illustration des Projekts – keine Original-Screenshots der virtuellen Maschine.</sub>

<br><br>

![Status](https://img.shields.io/badge/Status-erfolgreich%20getestet-29a36a?style=for-the-badge)
![Proxmox](https://img.shields.io/badge/Proxmox%20VE-9.2.3-E57000?style=for-the-badge)
![Commodore OS](https://img.shields.io/badge/Commodore%20OS-Vision%203.1-7854C8?style=for-the-badge)
![Desktop](https://img.shields.io/badge/MATE-Marco%20ohne%20Compositing-267BCE?style=for-the-badge)

**[📘 Vollständige Installationsdokumentation (PDF)](Commodore_OS_Proxmox_Installationsdokumentation-2.pdf)**

</div>

---

## 💡 Worum geht es?

In diesem Projekt wurde **Commodore OS Vision 3.1 (Release 9)** in einer virtuellen Maschine auf **Proxmox VE** installiert. Die Installation gelang, allerdings gab es ein hartnäckiges Problem mit der grafischen Oberfläche:

> Beim Schließen von Fenstern verschwanden teilweise Menüleiste, Dock und Desktopsymbole. Übrig blieben Hintergrundbild und Mauszeiger.

Der im Test funktionierende Weg war, die automatische Rückkehr zu **Compiz** zu unterbinden und stattdessen den MATE-Fenstermanager **Marco ohne Compositing** zu verwenden. Nach dem Speichern der Einstellungen wurde der funktionierende Zustand bestätigt.

Diese README vermittelt die wichtigsten Schritte. Die ausführliche Anleitung mit den Zwischenschritten, Fehlerbildern und Hinweisen liegt als [PDF im Repository](Commodore_OS_Proxmox_Installationsdokumentation-2.pdf).

## 🖥️ Verwendete virtuelle Maschine

| Komponente | Dokumentierte Konfiguration |
| :-- | :-- |
| Hypervisor | Proxmox VE **9.2.3** |
| Gastbetriebssystem | Commodore OS **Vision 3.1, Release 9** |
| Installations-ISO | `CommodoreOS3_1-20260428_0617.iso` |
| VM | `111` – `COMMODORE-OS` |
| Arbeitsspeicher | **8 GiB** |
| Prozessor | **6 Kerne**, 1 Socket, CPU-Typ `host` |
| BIOS / Maschine | **OVMF (UEFI)** / `q35` |
| Virtuelle Festplatte | **100 GB**, SCSI |
| SCSI-Controller | `VirtIO SCSI single` |
| Netzwerk | `VirtIO` |
| Anzeige (Ausgangszustand) | `Standard VGA (std)` |

> **Hinweis:** Dies ist die tatsächlich dokumentierte Testkonfiguration, **keine allgemeine Mindestanforderung**. Ein Wechsel zu VirtIO-GPU wurde im Verlauf vorgeschlagen; der letztlich eingestellte Anzeigetyp ist nicht abschließend dokumentiert.

## 🚀 Installationsweg im Überblick

### 1. Live-System starten und Oberfläche wiederherstellen

Die Commodore-OS-ISO in das virtuelle CD/DVD-Laufwerk einbinden und davon starten. Falls ein schwarzer Bildschirm oder ein LightDM-Fehler erscheint, in der **Konsole der virtuellen Maschine** mit `Strg` + `Alt` + `F2` (alternativ `F3`) in die Textkonsole wechseln.

Im dokumentierten Live-System wurde zunächst folgender Software-Grafik-Startversuch durchgeführt:

```bash
echo 'export LIBGL_ALWAYS_SOFTWARE=true' >> ~/.xsessionrc
sudo systemctl restart lightdm
```

Anschließend bei Bedarf mit `Strg` + `Alt` + `F7` zur grafischen Oberfläche zurückkehren. Das **Welcome-Fenster minimieren statt schließen**, solange die Installation noch läuft.

### 2. Terminal und Installer öffnen

Falls das Terminal nicht über den Desktop erreichbar ist, wurde es aus der Textkonsole so gestartet:

```bash
DISPLAY=:0 XAUTHORITY=/home/Commodore/.Xauthority \
mate-terminal --disable-factory
```

Eine verschwundene Menüleiste ließ sich im grafischen Terminal wiederherstellen:

```bash
mate-panel --replace
```

Danach den Installer aufrufen:

```bash
sudo minstall
```

Die richtige **virtuelle Zielfestplatte** im Installer prüfen und die Installation abschließen. Anschließend die VM von der installierten Festplatte starten.

### 3. Compiz nicht mehr automatisch erzwingen

Im installierten System trat das Problem beim Schließen von Fenstern erneut auf. Entscheidend war eine vom Commodore-System ausgewertete Sperrdatei. In der Textkonsole der **VM**:

```bash
sudo touch /usr/share/commodoreos/flags/disable_force_compiz.flg
```

**Auf den genauen Dateinamen achten:** `disable_force_compiz.flg` enthält Unterstriche und keine Bindestriche.

### 4. Marco testweise statt Compiz starten

Der erfolgreiche Testaufruf lautete:

```bash
nohup env DISPLAY=:0 \
XAUTHORITY=/home/Commodore/.Xauthority \
marco --replace --no-composite --sm-disable \
> /tmp/marco-test.log 2>&1 &
```

Dann prüfen, ob Marco läuft:

```bash
pgrep -a -x marco
```

Nach Rückkehr zur Oberfläche mit `Strg` + `Alt` + `F7` ließen sich im Test mehrere Fenster öffnen und schließen, ohne dass Leiste oder Dock verschwanden.

### 5. Funktionierende Einstellung dauerhaft speichern

Die folgenden Befehle wurden **im grafischen Terminal als normal angemeldeter Benutzer, ohne `sudo`**, ausgeführt:

```bash
gsettings set org.mate.session.required-components windowmanager 'marco'
gsettings set org.mate.Marco.general compositing-manager false
```

Danach das System neu starten und erneut verschiedene Fenster schließen. Der erfolgreiche Zustand wurde anschließend bestätigt.

> ⚠️ **Wichtig:** Sämtliche hier aufgeführten Linux-Befehle gehören **in die Commodore-OS-VM**, nicht in die Shell des Proxmox-Hosts. Die `XAUTHORITY`-Pfade beziehen sich auf den im Test verwendeten Benutzer `Commodore` und müssen bei anderen Benutzern angepasst werden.

## 🧪 Ergebnis

| Prüfung | Ergebnis |
| :-- | :-- |
| Installation von Commodore OS Vision 3.1 | ✅ Abgeschlossen |
| Start des installierten Systems | ✅ Erfolgreich |
| Fenster öffnen und schließen | ✅ Funktioniert mit Marco |
| Leiste, Dock und Desktop beim Schließen erhalten | ✅ Im Funktionstest bestätigt |
| Marco als gewünschter Fenstermanager gespeichert | ✅ Durchgeführt |
| Langzeitstabilität / umfangreicher Belastungstest | ⏳ Nicht separat dokumentiert |
| Updates und Neuinstallation auf einer zweiten VM | ⏳ Nicht separat dokumentiert |

**Abwägung:** Die Stabilität wurde durch den Wechsel von Compiz zu Marco **ohne Compositing** erreicht. Auf die ursprünglichen Compiz-Effekte wurde dabei verzichtet. Die vollständige technische Ursache des Grafikproblems ist nicht abschließend nachgewiesen.

## 🔎 Nützliche Prüf- und Diagnosebefehle

```bash
# Gespeicherten Fenstermanager prüfen
gsettings get org.mate.session.required-components windowmanager

# Compositing-Einstellung prüfen
gsettings get org.mate.Marco.general compositing-manager

# Laufende Fenstermanager prüfen
pgrep -a -x marco
pgrep -a -x compiz

# Bei erneuten Problemen: Sitzung und Protokolle
pgrep -a -f 'mate-session|mate-panel|compiz|cairo-dock|marco'
tail -n 60 ~/.xsession-errors
sudo journalctl -b -u lightdm --no-pager -n 40
```

Diese Befehle dienen der **erneuten Kontrolle**; die Durchführung sämtlicher Abfragen nach dem letzten Neustart wurde nicht separat protokolliert.

## 📂 Repository-Inhalt

```text
commodore-os-proxmox/
├── README.md
├── Commodore_OS_Proxmox_Installationsdokumentation-2.pdf
└── assets/
    └── commodore-os-collage.png
```

## 📚 Dokumentation und Links

- **[Vollständige Projektdokumentation (PDF)](Commodore_OS_Proxmox_Installationsdokumentation-2.pdf)** – der im Versuch funktionierende Weg mit allen 13 Schritten und Fehleranalysen.
- [Proxmox VE](https://www.proxmox.com/de/produkte/proxmox-virtual-environment/uebersicht)
- [Mesa – Umgebungsvariablen](https://docs.mesa3d.org/envvars.html)
- [Marco – Handbuchseite](https://manpages.debian.org/unstable/marco/x-window-manager.1.en.html)

## 👤 Projekt

**Andreas Schauer** · Privates Virtualisierungs- und Retro-Computing-Projekt  
**Dokumentationsstand:** 8. Oktober 2026

Dieses Repository dokumentiert eigene Tests und Lösungen. Es ist **kein offizielles Projekt von Commodore OS oder Proxmox**. Produktnamen und Marken gehören den jeweiligen Inhabern.
