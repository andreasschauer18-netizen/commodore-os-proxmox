cd ~/GitHub-Profil

python3 - <<'PY'
from pathlib import Path

datei = Path("README.md")
text = datei.read_text(encoding="utf-8")

alt = """🔗 **[Multi-PI auf GitHub ansehen](https://github.com/andreasschauer18-netizen/Multi-PI)**

*Hinweis: Das Multi-PI-Repository ist privat.
Der Quellcode ist daher nicht öffentlich zugänglich.*"""

neu = "🔒 **GitHub-Repository: Privat**"

if alt not in text:
    raise SystemExit("Textstelle nicht gefunden – keine Änderung.")

datei.write_text(text.replace(alt, neu, 1), encoding="utf-8")
print("Multi-PI-Darstellung vereinheitlicht.")
PY

git diff --check
git diff -- README.md

git add README.md
git commit -m "Private Repository-Kennzeichnung vereinheitlicht"
git push origin main
