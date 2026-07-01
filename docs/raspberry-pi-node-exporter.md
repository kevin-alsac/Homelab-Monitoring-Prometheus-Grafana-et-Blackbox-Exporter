# Installation de node_exporter sur Raspberry Pi

Ce document décrit l'installation de `node_exporter` sur un Raspberry Pi 64-bit.

## Pré-requis

Vérifier l'architecture :

```bash
uname -m
```

Résultat attendu :

```text
aarch64
```

## Installation temporaire

```bash
mkdir -p ~/node_exporter-test
cd ~/node_exporter-test

VERSION="1.11.1"
wget https://github.com/prometheus/node_exporter/releases/download/v${VERSION}/node_exporter-${VERSION}.linux-arm64.tar.gz

tar xvf node_exporter-${VERSION}.linux-arm64.tar.gz
mv node_exporter-${VERSION}.linux-arm64 node_exporter

./node_exporter/node_exporter --web.listen-address=ADRESSE_IP_LOCALE_DU_RPI:9100
```

Test local :

```bash
curl http://localhost:9100/metrics | head
```

Test depuis Prometheus :

```text
http://IP_DU_RPI:9100/metrics
```

## Service systemd optionnel

Créer le service :

```bash
sudo nano /etc/systemd/system/node_exporter-test.service
```

Contenu :

```ini
[Unit]
Description=Node Exporter Test
After=network.target

[Service]
Type=simple
ExecStart=/home/UTILISATEUR/node_exporter/node_exporter --web.listen-address=ADRESSE_IP_LOCALE_DU_RPI:9100
Restart=unless-stopped

[Install]
WantedBy=multi-user.target
```

Activer :

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter-test
sudo systemctl status node_exporter-test
```

Suppression :

```bash
sudo systemctl stop node_exporter-test
sudo systemctl disable node_exporter-test
sudo rm /etc/systemd/system/node_exporter-test.service
sudo systemctl daemon-reload
sudo systemctl reset-failed
rm -rf ~/node_exporter-test
```
