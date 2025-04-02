Traefik-dev-local
=================

Create docker compose settings to use traefik on all local projects

Documentation : https://doc.traefik.io/traefik/

Pour démarrer, il faut : 
- Créer les certificats ssl `make generate-ssl-certificate`
- Créer et demarrer l'instance docker `make docker-init`

Puis rendez-vous sur le dashboard de traefik http://localhost:8080/

Ensuite, il faut ajouter les labels suivants sur le container du projet que vous voulez voir apparaitre dans traefik

`touch compose-ovveride.yml`

```
services:
  nginx:
    networks:
      - traefik-proxy-network
    
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app1.rule=Host(`app1.local`)"
      - "traefik.http.routers.app1.tls=true"
      - "traefik.http.routers.app1.entrypoints=websecure"
      - "traefik.http.services.app1.loadbalancer.server.port=80"

networks:
  traefik-proxy-network:
    external: true
```