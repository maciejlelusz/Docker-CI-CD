# Docker CI/CD — Moduł 08  
## Automatyczne wdrażanie aplikacji na Docker Swarm za pomocą docker-compose + Jenkins

W tym module kończymy budowę prawdziwego pipeline’u CI/CD:  
po zbudowaniu obrazu Docker, Jenkins automatycznie wdroży go na klastrze **Docker Swarm** jako usługę.

Zrobimy to za pomocą:

- pliku **docker-compose.yml**,
- polecenia `docker stack deploy`,
- kroku shellowego w Jenkinsie,
- visualizera pokazującego działające usługi Swarm.

---

# 1. Utworzenie pliku `docker-compose.yml` w repozytorium (GitLab)

Przejdź na serwer GitLab do katalogu repozytorium **Cmentarna-Polka** i utwórz plik:

```yaml
version: '3.1'

services:
  app:
    image: [nazwa repo]/[obraz]:latest
    ports:
      - "8080:80"
    deploy:
      replicas: 2
    command: nginx -g 'daemon off;'
```

Wyjaśnienie:

- `image:` – obraz kontenera z Docker Hub, który Jenkins będzie aktualizował,
- `ports:` – aplikacja będzie dostępna na porcie 8080,
- `replicas: 2` – Swarm uruchomi dwa kontenery,
- `command:` – ścieżka startowa aplikacji (nginx w trybie „foreground”).

---

# 2. Utworzenie pliku `index.html`

W katalogu repozytorium:

```bash
echo "Hello Docker Swarm CI/CD!" > index.html
```

Możesz wpisać tam dowolną treść — będzie wyświetlana po wdrożeniu.

---

# 3. Aktualizacja pliku Dockerfile

Edytuj Dockerfile na wersję 0.2:

```Dockerfile
# Version: 0.2
FROM ubuntu:16.04
MAINTAINER Maciej Lelusz "maciej.lelusz@inleo.pl"

RUN apt-get update && apt-get install -y nginx

COPY index.html /var/www/html/index.html

EXPOSE 80
```

Różnice:

- dodajemy index.html jako część obrazu,
- zawartość strony staje się wersją aplikacji — pipeline będzie wdrażać nowsze wersje przy każdym commicie.

---

# 4. Konfiguracja kroku wdrożeniowego w Jenkins

Przejdź do:

```
http://jenkins:8080
```

Następnie:

```
Cmentarna-Polka-Deploy → Konfiguruj → Kroki budowania → Dodaj krok budowania → Execute shell
```

W kroku **Execute shell** dodaj:

```bash
export DOCKER_HOST="tcp://[manager01-wew-IP]:4243"

docker stack rm cmentarna-polka

docker stack deploy -c /var/lib/jenkins/workspace/Cmentarna-Polka-Deploy/docker-compose.yml cmentarna-polka
```

Wyjaśnienie:

- `export DOCKER_HOST` — Jenkins komunikuje się ze Swarm Managerem,
- `docker stack rm` — usuwa starą wersję stosu,
- `docker stack deploy` — wdraża nową wersję budowaną z Dockerfile.

`/var/lib/jenkins/workspace/Cmentarna-Polka-Deploy/` to domyślna ścieżka workspace projektu.

Zapisz konfigurację.

---

# 5. Odświeżenie Visualizera na manager01

Jeśli Visualizer działa, usuniemy go i odpalimy go ponownie z poprawnym DOCKER_HOST.

Na **manager01**:

```bash
docker service rm viz
```

Uruchom Visualizer ponownie:

```bash
sudo docker service create \
  --name=viz \
  --publish=8090:8080/tcp \
  --constraint=node.role==manager \
  -e DOCKER_HOST="[manager01-wew-IP]:4243" \
  dockersamples/visualizer
```

Wejdź w przeglądarkę:

```
http://manager01:8090
```

---

# 6. Wykonanie nowego push’a w repozytorium

Na serwerze GitLab, w katalogu repozytorium:

```bash
git add . && git commit -m "change" && git push -u origin master
```

Po tym:

1. GitLab wywoła webhook → Jenkins  
2. Jenkins:
   - zbuduje obraz Docker,
   - wypchnie go na Docker Hub,
   - wykona `docker stack deploy`, tworząc nową wersję aplikacji  
3. Visualizer pokaże aktualizujące się kontenery Swarm

To pełny pipeline CI/CD 🎉

---

# Podsumowanie

W tym module:

- dodaliśmy docker-compose do repozytorium jako „manifest wdrożeniowy”,
- zaktualizowaliśmy aplikację oraz Dockerfile,
- dodaliśmy krok wdrożeniowy w Jenkins,
- uruchomiliśmy automatyczne deploymenty na Docker Swarm,
- odświeżyliśmy narzędzie Visualizer do podglądu klastra.

To oznacza, że masz już **pełne CI/CD**:

> Git push → GitLab → Jenkins → Docker Build → Docker Hub → Docker Swarm → Wdrożenie 🚀

Możesz rozwijać pipeline, dodawać testy, staging, monitoring i rolling updates.
