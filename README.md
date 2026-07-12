# shared-requirements

Zentrale Versionsverwaltung für mehrere Bot-Repos über ein
[pip Constraints-File](constraints.txt).

## Funktionsweise

Jeder Bot behält seine **eigene** `requirements.txt` und entscheidet dort
selbst, welche Pakete er braucht (z. B. `discord.py` oder `nextcord` — beide
können nebeneinander existieren, ohne Konflikt). Nur die **Versionen**
dieser Pakete kommen zentral aus [constraints.txt](constraints.txt).

1. **Zentrale Versionspins pflegen:** In [constraints.txt](constraints.txt)
   stehen Paketname + Version, z. B. `discord.py==2.4.0`. Diese Datei
   installiert selbst nichts — sie pinnt nur die Version, *falls* ein Bot
   das jeweilige Paket überhaupt anfordert. Fordert ein Bot ein Paket an,
   das hier nicht auftaucht, greift einfach keine Pin (kein Fehler).

2. **In jedem Bot-Repo einbinden:** Die `requirements.txt` des Bots verweist
   per `-c` (constraints) auf diese Datei und listet darunter die vom Bot
   tatsächlich benötigten Pakete — ohne Version:

   ```
   # Bot 1 – requirements.txt
   -c https://raw.githubusercontent.com/DEIN-USER/shared-requirements/main/constraints.txt
   discord.py
   requests
   python-dotenv
   ```

   ```
   # Bot 2 – requirements.txt
   -c https://raw.githubusercontent.com/DEIN-USER/shared-requirements/main/constraints.txt
   nextcord
   requests
   ```

   `pip install -r requirements.txt` installiert damit genau die vom Bot
   gewählten Pakete, aber in der Version, die zentral in `constraints.txt`
   gepflegt wird. Das Dockerfile der Bots muss dafür nicht geändert werden.

3. **Automatischer Redeploy:** Der Workflow
   [.github/workflows/trigger-redeploys.yml](.github/workflows/trigger-redeploys.yml)
   ruft bei jedem Push auf `main`, der `constraints.txt` ändert, die
   Dokploy-Redeploy-Webhooks aller Bot-Apps auf.

## Einrichtung

1. Dieses Verzeichnis als eigenes GitHub-Repo `shared-requirements`
   veröffentlichen (public, sofern die Paketliste unkritisch ist — dann
   funktioniert der Rohdatei-Link ohne Auth-Token).
2. Für jede Bot-App in Dokploy den Deploy-Webhook kopieren
   (App → Settings → Webhook) und als GitHub-Secret in diesem Repo anlegen,
   z. B. `DOKPLOY_WEBHOOK_BOT1`, `DOKPLOY_WEBHOOK_BOT2`, `DOKPLOY_WEBHOOK_BOT3`.
3. Im Workflow die Anzahl/Namen der Steps an die tatsächlichen Bots anpassen.
4. Falls das Repo **privat** bleiben soll: In den Bot-Repos statt der reinen
   Raw-URL eine Variante mit Token verwenden, z. B.
   `-c https://<TOKEN>@raw.githubusercontent.com/DEIN-USER/shared-requirements/main/constraints.txt`.
