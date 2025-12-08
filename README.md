Traefik-dev-local
=================

Create docker compose settings to use traefik on all local projects

Documentation : https://doc.traefik.io/traefik/

## Prérequis

- Docker et Docker Compose installés
- Make installé
- mkcert installé (sera utilisé par `make mkcert`)

## Structure

```
traefik-dev-local/
├── compose.yml
├── Makefile
├── traefik/
│   ├── traefik.yml (configuration principale de traefik)
│   └── tls.yml (configuration TLS de traefik)
└── certs/
    └── (certificats générés)
```

## Utilisation

Pour démarrer, il faut : 
- Créer les certificats ssl `make mkcert` : cette commande va créer des certificats wildcard "*.dev.local" avec l'outils [mkcert](https://github.com/FiloSottile/mkcert)
- Créer et demarrer l'instance docker `make docker-init`

Ajouter les liens dans le fichier `/etc/hosts` :

```
127.0.0.1 traefik.dev.local mail.dev.local
127.0.0.1 app1.dev.local
# Ajouter ici tous vos autres domaines *.dev.local
```

Puis rendez-vous sur le dashboard de traefik https://traefik.dev.local

## Bonus :

Cette configuration docker compose contient possède un container mailpit pour intercepter les mails envoyés par vos applications en dev local.
Rendez-vous sur http://mailpit.dev.local pour voir les mails interceptés.

## Connecter un projet à Traefik

Ensuite, il faut ajouter les labels suivants sur le container du projet que vous voulez voir apparaitre dans traefik et il faut que les containers soient également dans le même network que traefik

Pour celà, il faut aller dans le dossier de votre projet et faire `touch compose.override.yml`

```
services:
  service-web-app1:
    networks:
      - traefik-proxy-network
    
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app1.rule=Host(`app1.dev.local`)"
      - "traefik.http.routers.app1.tls=true"
      - "traefik.http.routers.app1.entrypoints=websecure"
      - "traefik.http.services.app1.loadbalancer.server.port=80"

networks:
  traefik-proxy-network:
    external: true
```

Autre astuce, si vous avez plusieurs containers connectés à traefik qui ont besoin de communiquer entre eux (par exemple, un frontend qui appelle une API), il faut définir les alias sur le network du container traefik. Comme ça, depuis chaque container tous les hosts internes pointeront vers traefik qui fera ensuite le routing.

```
      proxy:
        aliases:
          - api-app1.dev.local
          - site1.dev.local
          - site2.dev.local
          - app-test.dev.local
```

**Exemple d'usage** : Votre app `site1.dev.local` peut maintenant faire des requêtes vers `https://api-app1.dev.local` et Traefik routera automatiquement vers le bon container.

## Dépannage

**Le container n'apparaît pas dans Traefik :**
- Vérifier que le container est bien dans le network `traefik-proxy-network`
- Vérifier les labels du container
- Consulter les logs : `make docker-logs-full`

**Erreur de certificat SSL :**
- Régénérer les certificats : `make mkcert` (cette commande installe automatiquement l'autorité de certification locale)
- Redémarrer votre navigateur après la génération des certificats
- Sur certains navigateurs (Firefox notamment), il peut être nécessaire d'importer manuellement le certificat CA de mkcert
