# Docker CI/CD — Moduł 02  
## Instalacja Dockera + konfiguracja Docker Swarm

W tym module przygotujemy środowisko kontenerowe dla CI/CD:

- zainstalujemy Dockera na trzech maszynach:
  - **manager01**
  - **worker01**
  - **jenkins**
- skonfigurujemy **Docker Swarm** (manager + worker),
- uruchomimy **Visualizer**, aby podglądać klaster w przeglądarce.

---

# Instalacja Dockera (na manager01, worker01 i jenkins)

Poniższe kroki wykonaj **na każdej z trzech maszyn**.

## 1. Aktualizacja systemu i instalacja zależności

```bash
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
```

---

## 2. Dodanie repozytorium Docker CE

Pobieramy klucz GPG:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Dodajemy repozytorium Dockera:

```bash
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Aktualizacja repozytoriów:

```bash
sudo apt-get update
```

---

## 3. Instalacja Docker CE

```bash
sudo apt-get -y install docker-ce
```

Weryfikacja działania:

```bash
sudo docker info
```

Jeśli widzisz informacje o silniku Docker — instalacja powiodła się.

---

# Konfiguracja klastra Docker Swarm

Docker Swarm umożliwia tworzenie klastrów z wieloma węzłami, skalowanie usług i zarządzanie kontenerami z poziomu jednego managera.

---

# Konfiguracja na węźle manager01

## 1. Odczyt adresu IP

```bash
ifconfig
```

Warto przepisać adres z interfejsu `eth0` lub `ens…`.

## 2. Inicjalizacja klastra Swarm

```bash
sudo docker swarm init --advertise-addr [IP]
```

Polecenie:

- inicjuje klaster,
- ustawia bieżący węzeł jako **manager**,
- generuje komendę `docker swarm join` dla workerów.

## 3. Weryfikacja

```bash
sudo docker info
sudo docker node ls
```

`docker node ls` powinno wyświetlić jeden węzeł z rolą `Leader`.

---

# Konfiguracja na węźle worker01

Użyj komendy przekazanej przez managera:  
(np. wygląda ona tak)

```bash
sudo docker swarm join --token <TOKEN> <IP_MANAGERA>:2377
```

Po dołączeniu sprawdź status:

```bash
sudo docker info
```

Na maszynie manager01 możesz potwierdzić:

```bash
sudo docker node ls
```

Powinieneś zobaczyć worker01 jako `Ready`.

---

# Uruchomienie Visualizera klastra

Visualizer pozwala podejrzeć węzły, kontenery i usługi klastra Swarm w wygodny sposób przez przeglądarkę.

Uruchom Visualizera na **manager01**:

```bash
sudo docker service create \
  --name=viz \
  --publish=8090:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer
```

---

## Weryfikacja działania

Lista usług:

```bash
sudo docker service ls
```

Lista kontenerów:

```bash
sudo docker container ls
```

---

# Dostęp do Visualizera

Wejdź w przeglądarce:

```
http://manager01:8090
```

Powinieneś zobaczyć graficzne przedstawienie klastra Swarm, z węzłami `manager01` oraz `worker01`.

---

# Podsumowanie

W tym module:

- zainstalowaliśmy Dockera na wszystkich maszynach,
- stworzyliśmy klaster Docker Swarm,
- dołączyliśmy worker nodes,
- uruchomiliśmy narzędzie do wizualizacji klastra.

Nasza architektura Docker CI/CD jest gotowa do budowania pipeline’ów!  
W kolejnym module skonfigurujemy **GitLab + GitLab Runner**.
