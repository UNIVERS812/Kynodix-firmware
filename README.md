# Binaires OTA Sensora

Dépôt **public** contenant uniquement les binaires de firmware (`*.bin`) servis
aux boîtiers ESP32 du parc Sensora. Il est séparé du dépôt privé
[`sensora-iot`](https://github.com/UNIVERS812/sensora-iot) parce que les assets
d'une release GitHub **privée** ne sont pas téléchargeables anonymement : un
ESP32 n'a pas de jeton d'authentification.

Le code source, le contrat d'interface et la documentation vivent dans
`sensora-iot`. Ici, il n'y a que des artefacts.

## Règle : un produit = un binaire

| Fichier | Produit (`DeviceType`) | Carte |
|---|---|---|
| `sensora-weather_esp32-vX.Y.Z.bin` | `weather_station` | `esp32dev` |
| `sensora-air_esp32-vX.Y.Z.bin` | `air_quality` | `esp32dev` |

Le nom encode déjà le produit **et** la carte : un binaire ne peut pas être
confondu avec un autre. Le boîtier vérifie en plus, avant tout téléchargement,
que le manifeste reçu annonce bien son `product_key` et son `board`.

## Arborescence

```text
sensora-firmware/
├── README.md
└── <version>/
    ├── sensora-weather_esp32-v<version>.bin
    ├── sensora-air_esp32-v<version>.bin
    └── SHA256SUMS.txt
```

## Publier une version

Depuis le dépôt `sensora-iot` :

```powershell
cd firmware
# 1. Mettre à jour FIRMWARE_VERSION dans include/version.h
# 2. Construire et calculer les empreintes
.\scripts\release.ps1 -Version 1.2.0 -All
```

Le script produit `firmware/dist/sensora-<env>-v<version>.bin` et affiche la
taille et le SHA-256 de chaque binaire. Puis :

```powershell
$version = "1.2.0"
New-Item -ItemType Directory -Force "../sensora-firmware/$version" | Out-Null
Copy-Item "dist/*.bin" "../sensora-firmware/$version/"
Get-FileHash "../sensora-firmware/$version/*.bin" -Algorithm SHA256 |
    ForEach-Object { "$($_.Hash.ToLower())  $(Split-Path $_.Path -Leaf)" } |
    Set-Content "../sensora-firmware/$version/SHA256SUMS.txt"
```

Committer, pousser, puis créer le tag `fw-v<version>` :

```bash
git add . && git commit -m "firmware: v1.2.0"
git tag fw-v1.2.0
git push origin main --tags
```

## URL à coller dans la plateforme

Dans `FirmwareRelease.file_url` (admin → Firmware OTA) :

```text
https://raw.githubusercontent.com/UNIVERS812/sensora-firmware/main/<version>/sensora-<env>-v<version>.bin
```

Renseigner également `sha256`, `size_bytes`, `channel` et le `DeviceType`
correspondant. L'URL doit être en **HTTPS** : le firmware refuse le HTTP clair.

> Le contenu de ce dépôt n'est pas secret (il est public par construction).
> Aucune clé d'API, aucun identifiant Wi-Fi ni secret de provisioning ne doit y
> être déposé.
