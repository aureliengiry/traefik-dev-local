Traefik-dev-local
=================

Create docker compose settings to use traefik on all local projects

Documentation : https://doc.traefik.io/traefik/

Pour démarrer, il faut : 
- Créer les certificats ssl `make generate-ssl-certificate`
- Créer et demarrer l'instance docker `make docker-init`

Puis rendez-vous sur le dashboard de traefik http://localhost:8080/

Ensuite, il faut ajouter les labels suivants sur le container du projet que vous voulez voir apparaitre dans traefik

`touch compose.override.yml`

```
services:
  nginx-app1:
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

Pour que les containers puissent accéder aux containers en https, il faut créer des alias sur le container traefik

Exemple : 
```
services:
  traefik:
    networks:
      proxy: 
        aliases:
          - www.app1.local
          - www.app2.local
          - www.app3.local
```
De cette façon, l'app1 peut appeler l'app2 via https en passant par le proxy