# Docker CI/CD — Moduł 07  
## Zdalne wdrażanie kontenerów z Jenkinsa na Docker Swarm (manager01)

W tym module wykonamy kluczowy krok w budowie pełnego pipeline’u CI/CD:

- włączymy **zdalny dostęp do Docker Daemon** na węźle `manager01`,
- skonfigurujemy Jenkins, aby łączył się z Dockerem po TCP,
- uruchomimy pipeline i zobaczymy, jak Jenkins wdraża kontenery bezpośrednio do klastra Swarm.

---

# 1. Konfiguracja Docker Daemon na manager01

Domyślnie Docker działa tylko lokalnie i nasłuchuje na gnieździe UNIX.  
Aby Jenkins mógł wysyłać polecenia `docker build`, `docker service update`, `docker run` itd. — musimy udostępnić Docker API przez TCP.

⚠️ **Uwaga:** To połączenie nie jest domyślnie szyfrowane. W środowisku produkcyjnym należy włączyć TLS.  
Tu konfigurujemy środowisko warsztatowe.

---

## Edycja pliku docker.service

Na węźle **manager01**:

```bash
sudo vi /lib/systemd/system/docker.service
```

Znajdź linię rozpoczynającą się od:

```
ExecStart=/usr/bin/dockerd -H...
```

I zamień ją na:

```
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243
```

Dzięki temu Docker będzie nasłuchiwał na wszystkich interfejsach na porcie **4243**.

---

## Eksport zmiennej środowiskowej (opcjonalne)

```bash
sudo export DOCKER_HOST="tcp://0.0.0.0:4243"
```

---

## Restart usługi Docker

```bash
sudo systemctl daemon-reload
sudo service docker restart
```

---

## Weryfikacja działania

```bash
sudo docker ps
```

Powinno działać jak wcześniej, ale dodatkowo port *4243* jest już otwarty na zdalne połączenia.

---

# 2. Konfiguracja Jenkinsa — Docker Host URI

Wejdź do panelu:

```
http://jenkins:8080
```

Przejdź do:

```
Cmentarna-Polka-Deploy → Konfiguruj → Kroki budowania
```

Znajdź krok **Docker Build and Publish** i ustaw:

```
Docker Host URI: tcp://[manager-ip]:4243
```

Przykład:

```
tcp://192.168.50.10:4243
```

Dzięki temu Jenkins będzie:

- budował obrazy kontenerów,
- publikował je na Docker Hub,
- oraz (w kolejnych modułach) wdrażał je bezpośrednio na Swarm.

Zapisz zmiany.

---

# 3. Test działania — uruchom pipeline

Uruchom job:

```
Cmentarna-Polka-Deploy → Build Now
```

W tym czasie połącz się z manager01 i obserwuj kontenery:

```bash
sudo watch docker ps
```

Powinieneś zobaczyć kontenery:

- budowane przez Jenkinsa,
- publikowane do Docker Hub,
- a w kolejnych modułach także wdrażane jako usługi Swarm.

---

# Podsumowanie

W tym module:

- udostępniliśmy Docker API na porcie TCP,
- skonfigurowaliśmy Jenkinsa do zdalnej komunikacji z Docker Swarm,
- wykonaliśmy pierwszy zdalny build przy użyciu Docker Host URI.

Od tego momentu Jenkins może zarządzać klastrem Swarm w pełni automatycznie — to fundament pełnego CI/CD.

W kolejnym kroku zbudujemy pipeline, który **automatycznie wdroży aplikację jako usługę Docker Swarm**.
