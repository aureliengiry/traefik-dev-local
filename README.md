Traefik Dev Local
=================

Stack Docker locale avec **Traefik** comme reverse proxy pour exposer facilement vos projets en développement avec HTTPS.

## 📋 Services inclus

| Service | URL | Port | Documentation |
|---------|-----|------|---------------|
| **Traefik** | https://traefik.dev.local | 80, 443 (dashboard: 8080) | [doc.traefik.io](https://doc.traefik.io/traefik/) |
| **Mailpit** | https://mailpit.dev.local | 8025 | [github.com/axllent/mailpit](https://github.com/axllent/mailpit) |
| **Dozzle** | https://dozzle.dev.local | 8080 | [github.com/amir20/dozzle](https://github.com/amir20/dozzle) |

### Services optionnels (décommenter dans `compose.yml`)

| Service | URL | Port | Documentation |
|---------|-----|------|---------------|
| Loki | https://loki.dev.local | 3100 | [grafana.com/oss/loki](https://grafana.com/oss/loki/) |
| Promtail | - | - | [grafana.com/docs/loki](https://grafana.com/docs/loki/latest/) |
| Grafana | https://grafana.dev.local | 3000 | [grafana.com/docs](https://grafana.com/docs/) |
| Prometheus | https://prometheus.dev.local | 9090 | [prometheus.io/docs](https://prometheus.io/docs/introduction/overview/) |
| Alertmanager | https://alertmanager.dev.local | 9093 | [prometheus.io/docs/alerting](https://prometheus.io/docs/alerting/latest/alertmanager/) |
| Node Exporter | https://node-exporter.dev.local | 9100 | [github.com/prometheus/node_exporter](https://github.com/prometheus/node_exporter) |

---

## ⚙️ Prérequis

- Docker & Docker Compose
- make
- mkcert (pour les certificats SSL locaux)
- Ajouter les DNS dans `/etc/hosts` (voir [Configuration DNS](#-configuration-dns))

---

## 🚀 Installation

### 1. Cloner le dépôt
```bash
git clone <url-du-depot>
cd traefik-dev-local
```

### 2. Configurer les domaines SSL
Éditer `.env` et définir `MKCERT_DOMAINS` avec vos domaines locaux :
```env
MKCERT_DOMAINS="*.dev.local localhost 127.0.0.1 ::1"
```

### 3. Générer les certificats
```bash
make mkcert
```

### 4. Démarrer la stack
```bash
make docker-init
```

La stack est accessible à : https://traefik.dev.local

---

## 📧 Configuration DNS

Ajouter dans `/etc/hosts` :
```
127.0.0.1 traefik.dev.local
127.0.0.1 mailpit.dev.local
127.0.0.1 dozzle.dev.local
```

Pour les services optionnels, ajouter également :
```
127.0.0.1 loki.dev.local
127.0.0.1 grafana.dev.local
127.0.0.1 prometheus.dev.local
127.0.0.1 alertmanager.dev.local
127.0.0.1 node-exporter.dev.local
```

---

## 📦 Ajouter un nouveau projet

Pour exposer un container via Traefik, ajouter les labels suivants dans votre `compose.yml` (ou via un fichier `compose.override.yml`):

```yaml
services:
  mon-app:
    networks:
      - traefik-proxy-network
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.mon-app.rule=Host(`mon-app.dev.local`)"
      - "traefik.http.routers.mon-app.tls=true"
      - "traefik.http.routers.mon-app.entrypoints=websecure"
      - "traefik.http.services.mon-app.loadbalancer.server.port=80"

networks:
  traefik-proxy-network:
    external: true
```

> ⚠️ **Important** : Le réseau `traefik-proxy-network` doit être déclaré comme `external: true`.

### Ajouter des alias Traefik (optionnel)

Pour que vos containers puissent accéder à d'autres services en HTTPS via Traefik, ajouter des alias dans `compose.override.yml` :

```yaml
services:
  traefik:
    networks:
      proxy:
        aliases:
          - mon-app.dev.local
          - api.mon-app.dev.local
```

---

## 🛠️ Commandes utiles

| Commande | Description |
|----------|-------------|
| `make docker-init` | Démarrer toute la stack |
| `make docker-start` | Démarrer les containers |
| `make docker-stop` | Arrêter les containers |
| `make docker-restart` | Redémarrer la stack |
| `make docker-logs` | Voir les logs (suivi) |
| `make docker-logs-full` | Voir tous les logs depuis le début |
| `make mkcert` | Régénérer les certificats SSL |

---

## 📊 Monitoring & Logs

- **Dozzle** : Interface web pour visualiser les logs de tous vos containers → https://dozzle.dev.local
- **Traefik Dashboard** : Tableau de bord Traefik → https://traefik.dev.local
- **Mailpit** : Serveur SMTP local pour tester l'envoi d'emails → https://mailpit.dev.local

---

## 🔧 Troubleshooting

### Erreur de certificat SSL
```bash
# Régénérer les certificats
make mkcert
# Redémarrer la stack
make docker-restart
```

### Container non accessible
- Vérifiez que le container est bien sur le réseau `traefik-proxy-network`
- Vérifiez les labels Traefik
- Vérifiez que le DNS est bien configuré dans `/etc/hosts`
- Consultez les logs avec `make docker-logs`
