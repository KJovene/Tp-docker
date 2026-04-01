# TP Docker - Mini plateforme multi-conteneurs

Ce repo livre une stack Docker Compose conforme au TP : reverse proxy Nginx, service de test whoami, supervision Uptime Kuma, visualisation des logs avec Dozzle, et Redis en service interne.

## Arborescence

```text
tp-plateforme-docker/
├── compose.yml
└── nginx/
	├── conf.d/
	│   └── default.conf
	└── html/
		└── index.html
```

## Services

- `nginx` : point d'entree principal sur `http://localhost:8080`
- `whoami` : backend de test accessible via `http://localhost:8080/whoami`
- `uptime-kuma` : supervision sur `http://localhost:3001`
- `dozzle` : visualisation des logs Docker sur `http://localhost:9999`
- `redis` : service interne uniquement sur `back_net`

## Architecture reseau

- `front_net` : `nginx`, `whoami`, `uptime-kuma`, `dozzle`
- `back_net` : `whoami`, `redis`, `uptime-kuma`

Regles respectees :

- `redis` uniquement sur `back_net`
- `nginx` uniquement sur `front_net`
- `whoami` sur `front_net` et `back_net`

## Volumes persistants

- `kuma_data` : donnees de supervision Uptime Kuma
- `redis_data` : persistance Redis
- `nginx_logs` : journaux Nginx

## Lancement

Depuis le dossier `tp-plateforme-docker` :

```powershell
docker compose up -d
```

Arret :

```powershell
docker compose down
```

Arret en conservant la persistance :

```powershell
docker compose down
docker compose up -d
```

Suppression aussi des volumes :

```powershell
docker compose down -v
```

## Commandes de verification

Verifier les conteneurs :

```powershell
docker compose ps
docker ps
```

Verifier les reseaux :

```powershell
docker network ls
docker network inspect tp-plateforme-docker_front_net
docker network inspect tp-plateforme-docker_back_net
```

Verifier les volumes :

```powershell
docker volume ls
docker volume inspect tp-plateforme-docker_kuma_data
docker volume inspect tp-plateforme-docker_redis_data
docker volume inspect tp-plateforme-docker_nginx_logs
```

Verifier les acces web :

```powershell
curl http://localhost:8080/
curl http://localhost:8080/whoami
curl http://localhost:3001/
curl http://localhost:9999/
```

Verifier les logs :

```powershell
docker compose logs nginx
docker compose logs whoami
docker compose logs uptime-kuma
docker compose logs dozzle
```

Verifier Redis depuis le reseau interne :

```powershell
docker compose exec redis redis-cli ping
```

## Mise en service attendue

1. Lancer `docker compose up -d`.
2. Ouvrir `http://localhost:8080` pour la page d'accueil.
3. Ouvrir `http://localhost:8080/whoami` pour verifier le reverse proxy.
4. Ouvrir `http://localhost:3001` pour initialiser Uptime Kuma.
5. Ouvrir `http://localhost:9999` pour visualiser les logs dans Dozzle.

## Partie supervision

Dans Uptime Kuma, ajouter deux moniteurs HTTP :

- `nginx-home` avec l'URL `http://nginx`
- `whoami-proxy` avec l'URL `http://whoami`

Comme Uptime Kuma est connecte aux deux reseaux, il peut joindre les services internes par leur nom Docker.

## Partie logs

Dans Dozzle, filtrer sur :

- `nginx`
- `whoami`

Vous verrez les acces HTTP, les en-tetes et les informations de requete remontees par whoami.

## Test de panne

Arreter whoami :

```powershell
docker compose stop whoami
```

Verifier le comportement :

- `http://localhost:8080/` reste disponible
- `http://localhost:8080/whoami` renvoie une erreur `502 Bad Gateway`
- Uptime Kuma doit detecter la panne sur le moniteur cible

Redemarrer whoami :

```powershell
docker compose start whoami
```

## Durabilite

Tester la persistance :

1. Configurer Uptime Kuma une premiere fois.
2. Redemarrer la stack avec `docker compose down` puis `docker compose up -d`.
3. Verifier que la configuration Uptime Kuma est toujours presente.
4. Verifier que les volumes existent encore avec `docker volume ls`.

## Livrables

- `compose.yml`
- configuration Nginx
- `index.html`
- captures `docker ps`, `docker network ls`, `docker volume ls`
- capture Dozzle
- capture Uptime Kuma
- compte rendu technique : ce README

## Bonus integres

- politique `restart: unless-stopped` appliquee sur tous les services
