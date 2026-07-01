# Homelab Monitoring — Prometheus, Grafana et Blackbox Exporter

Projet de supervision open source pour un homelab personnel.

L'objectif est de centraliser dans Grafana l'état de plusieurs équipements du réseau local :

- Raspberry Pi avec `node_exporter`
- NAS TerraMaster avec tests ICMP, HTTP et SMB
- Box Internet avec test ICMP
- Services réseau internes
- Prometheus pour la collecte des métriques
- Grafana pour les dashboards
- Blackbox Exporter pour les tests de disponibilité

> Projet réalisé dans un environnement de test local avec Docker.

---

## Architecture

```text
Machine Docker locale
└── Docker Compose
    ├── prometheus
    ├── grafana
    └── blackbox-exporter

Raspberry Pi
└── node_exporter :9100

NAS TerraMaster
├── ICMP ping
├── HTTP interface web
└── TCP 445 SMB

Box Internet
└── ICMP ping
```

---

## Objectifs du projet

Ce projet permet de pratiquer :

- Docker et Docker Compose
- Prometheus
- Grafana
- Exporters Prometheus
- Supervision réseau
- Supervision Linux
- Tests ICMP, HTTP et TCP
- Dashboards Grafana
- Organisation d'un projet GitHub technique

---

## Services utilisés

| Service           | Rôle                              |
| ----------------- | --------------------------------- |
| Prometheus        | Collecte des métriques            |
| Grafana           | Visualisation des métriques       |
| Blackbox Exporter | Tests réseau HTTP, ICMP, TCP      |
| Node Exporter     | Métriques système du Raspberry Pi |

---

## Équipements supervisés

### Raspberry Pi

Le Raspberry Pi expose ses métriques système via `node_exporter`.

Métriques disponibles :

- CPU
- RAM
- disque
- réseau
- charge système
- uptime
- métriques Linux générales

Endpoint Prometheus, à adapter selon le nom DNS ou l'adresse locale du Raspberry Pi :

```text
http://<rpi-host>:9100/metrics
```

### NAS TerraMaster

Le NAS est supervisé sans agent, avec Blackbox Exporter.

Tests configurés :

```text
ICMP : <nas-host>
HTTP : http://<nas-host>
TCP  : <nas-host>:445
```

Cela permet de vérifier :

- disponibilité réseau du NAS
- disponibilité de l'interface web
- disponibilité du service SMB
- latence réseau
- historique de coupures

### Box Internet

La box est supervisée avec un test ICMP :

```text
<box-host>
```

Cela permet de suivre :

- disponibilité de la passerelle
- latence locale
- coupures réseau

---

## Installation

### 1. Cloner le projet

```bash
git clone https://github.com/votre-utilisateur/homelab-monitoring.git
cd homelab-monitoring
```

### 2. Adapter les cibles surveillées

Modifier le fichier :

```bash
prometheus/prometheus.yml
```

Remplacer les placeholders selon votre réseau local :

```text
<rpi-host>  # Raspberry Pi avec node_exporter
<nas-host>  # NAS
<box-host>  # Box Internet ou passerelle locale
```

Exemple de cible Raspberry Pi dans `scrape_configs` :

```yaml
- job_name: "raspberry-pi"
  static_configs:
    - targets:
        - "<rpi-host>:9100"
```

### 3. Configurer le compte administrateur Grafana

Définir un mot de passe local avant de lancer la stack :

```bash
export GRAFANA_ADMIN_USER="<identifiant-admin-local>"
export GRAFANA_ADMIN_PASSWORD="<mot-de-passe-local-fort>"
```

Ne pas publier ces valeurs dans le dépôt.

### 4. Lancer la stack

```bash
docker compose up -d
```

### 5. Vérifier les conteneurs

```bash
docker ps
```

Services attendus :

```text
prometheus
grafana
blackbox-exporter
```

---

## Accès aux interfaces

### Grafana

```text
http://localhost:3000
```

Si l'accès via `localhost` ne fonctionne pas, utiliser l'adresse locale de la machine qui exécute Docker.

Identifiant administrateur :

```text
Fourni avec la variable d'environnement GRAFANA_ADMIN_USER
```

Le mot de passe doit être fourni avec la variable d'environnement `GRAFANA_ADMIN_PASSWORD`.

### Prometheus

```text
http://localhost:9090
```

Page des cibles :

```text
http://localhost:9090/targets
```

---

## Requêtes PromQL utiles

### Vérifier les cibles UP/DOWN

```promql
up
```

### Vérifier les tests Blackbox

```promql
probe_success
```

Résultat :

```text
1 = OK
0 = KO
```

### Latence des tests

```promql
probe_duration_seconds
```

### Code HTTP retourné par le NAS

```promql
probe_http_status_code
```

### CPU du Raspberry Pi

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{job="raspberry-pi",mode="idle"}[5m])) * 100)
```

### RAM utilisée du Raspberry Pi

```promql
100 - ((node_memory_MemAvailable_bytes{job="raspberry-pi"} / node_memory_MemTotal_bytes{job="raspberry-pi"}) * 100)
```

### Espace disque utilisé du Raspberry Pi

```promql
100 - ((node_filesystem_avail_bytes{job="raspberry-pi",mountpoint="/"} / node_filesystem_size_bytes{job="raspberry-pi",mountpoint="/"}) * 100)
```

### Uptime du Raspberry Pi

```promql
time() - node_boot_time_seconds{job="raspberry-pi"}
```

---

## Exemple de dashboard Grafana

Dashboard conseillé :

```text
Homelab Overview
├── Box Internet
│   ├── Ping OK/KO
│   └── Latence
├── NAS TerraMaster
│   ├── Ping OK/KO
│   ├── Interface web OK/KO
│   ├── SMB OK/KO
│   └── Latence
└── Raspberry Pi
    ├── CPU
    ├── RAM
    ├── Disque
    ├── Réseau
    └── Uptime
```

Types de panels recommandés :

```text
Stat        : états UP/DOWN du NAS et de la box
Gauge       : CPU, RAM et disque du Raspberry Pi
Time series : latence réseau et évolution CPU/RAM
Bar gauge   : comparaison rapide de plusieurs checks
Table       : diagnostic ponctuel
```

Exemples rapides :

```text
CPU Raspberry Pi      -> Gauge ou Time series
RAM Raspberry Pi      -> Gauge
Disque Raspberry Pi   -> Gauge
Uptime Raspberry Pi   -> Stat
Latence réseau        -> Time series avec probe_duration_seconds
Statut des services   -> Stat avec probe_success
```

---

## Sécurité

Pour un environnement de test local, les ports sont exposés sur la machine qui exécute Docker.

Pour un usage plus propre :

- ne pas exposer Prometheus publiquement
- ne pas exposer Blackbox Exporter publiquement
- limiter Grafana au réseau local ou à un VPN/Tailscale
- changer le mot de passe admin Grafana
- créer un compte Grafana en lecture seule
- ne pas ouvrir de port sur la box Internet
- éviter d'exposer des métriques contenant des informations sensibles

---

## Nettoyage

Arrêter la stack :

```bash
docker compose down
```

Supprimer aussi les volumes :

```bash
docker compose down -v
```

---

## Améliorations possibles

- Ajouter `windows_exporter` pour superviser le PC Windows
- Ajouter `snmp_exporter` pour obtenir plus d'informations du NAS
- Ajouter Alertmanager pour les alertes
- Ajouter Loki pour centraliser les logs
- Ajouter des dashboards Grafana versionnés en JSON
- Sécuriser l'accès avec Tailscale
- Déployer la stack sur un Raspberry Pi ou un mini-PC dédié

---

## État du projet

- Prometheus : fonctionnel
- Grafana : fonctionnel
- Blackbox Exporter : fonctionnel
- NAS : supervisé via ICMP, HTTP et TCP
- Box : supervisée via ICMP
- Raspberry Pi : supervisé via node_exporter dans l'architecture cible
