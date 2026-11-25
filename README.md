# Luwolf's Livestream Server — Releases

Official public repository for distributing builds and automatic updates for **Luwolf's Livestream Server** (`v1.1.6`).

- **Web App / Watch Streams:** [streams.luwolf.top](https://streams.luwolf.top/)
- **About the Author:** [luwolf.top](https://luwolf.top/)

## Download

- **[Download LLS-Installer.exe (Windows Online Installer)](https://github.com/Luwolf-Games/luwolf-s-livestream-server-app/raw/master/LLS-Installer.exe)**

Run `LLS-Installer.exe` (requests Administrator elevation automatically). It downloads the latest build from `luwolfs-livestream-server/`, installs it to `C:\Program Files\Luwolf's Livestream Server` (customizable), creates Desktop and Start Menu shortcuts, registers `lls-uninstaller.exe` in Windows Apps & Features, and launches the app.

## Repository Layout

- `LLS-Installer.exe`: online installer (bootstrapper). It always downloads and installs the latest contents of `luwolfs-livestream-server/` from this repository.
- `version.txt`: current published version (`1.1.6`).
- `luwolfs-livestream-server/`: latest application files (`luwolf-livestream-server.exe`, `lls-updater.exe`, `lls-uninstaller.exe`, `LICENSES/`, `README.md`, and `vendor/`).

## Auto-Update & Manual Update Checks

On launch, the application checks `version.txt` (and GitHub Releases) in this repository. Whenever a newer version is available, a mandatory update prompt is shown to download and install the latest build automatically. You can also check for updates manually anytime from the Home header or **Settings → About**.
