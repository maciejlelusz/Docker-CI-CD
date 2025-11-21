# Docker CI/CD — Moduł 06  
## Budowanie obrazów Docker i publikacja do Docker Hub za pomocą Jenkinsa

W tym module rozszerzymy nasz pipeline CI/CD o możliwość:

- budowania obrazów Docker,
- puszczania buildów automatycznie po pushu do GitLab,
- publikowania obrazów do Docker Hub,
- oznaczania ich numerem builda (`BUILD_NUMBER`).

Dzięki temu Jenkins będzie pełnić rolę automatycznego buildera i rejestru CI dla całego projektu.

---

# 1. Przygotowanie repozytorium na GitLab

Pracujemy w katalogu projektu: **Cmentarna-Polka**.

## Dodanie pliku `.dockerignore`

W katalogu repozytorium utwórz plik `.dockerignore`:

```bash
.git
```

Dzięki temu Docker podczas budowania obrazu nie będzie kopiował katalogu `.git`, co przyspieszy build i zmniejszy obraz.

---

## 2. Tworzenie Dockerfile

Utwórz plik `Dockerfile`:

```Dockerfile
# Version: 0.1
FROM ubuntu:16.04
MAINTAINER Imie Nazwisko "imie.nazwisko@adres.pl"

RUN apt-get update && apt-get install -y nginx
RUN echo 'Wujek Vernon, wujek Vernon.' > /var/www/html/index.html

EXPOSE 80
```

Ten obraz:

- instaluje Nginxa,
- tworzy stronę HTML z cytatem,
- otwiera port 80,
- jest bazą pod prosty serwis webowy.

---

# 3. Instalacja pluginu Docker Build and Publish w Jenkins

Przejdź pod adres:

```
http://jenkins:8080
```

Następnie:

```
Zarządzaj Jenkinsem → Zarządzaj wtyczkami → Dostępne
```

Znajdź plugin:

- **CloudBees Docker Build and Publish**

Zainstaluj:

```
→ Zainstaluj bez restartu
→ Uruchom ponownie kiedy wtyczka zostanie zainstalowana
```

---

# 4. Nadanie Jenkisowi uprawnień do Dockera

Jenkins musi mieć dostęp do Docker Daemon, aby budować obrazy.

Na maszynie Jenkins wykonaj:

```bash
sudo usermod -a -G docker jenkins
sudo usermod -a -G root jenkins
```

Następnie zrestartuj maszynę:

```bash
sudo reboot
```

Po restarcie Jenkins będzie mógł używać poleceń `docker`.

---

# 5. Konfiguracja joba w Jenkins do budowania i publikacji obrazu

Po restarcie:

```
http://jenkins:8080
```

Wejdź do projektu **Cmentarna-Polka-Deploy** → **Konfiguruj**.

Przewiń do sekcji:

```
Dodaj krok budowania → Docker Build and Publish
```

Ustaw:

- **Repository name:**  
  ```
  [użytkownik-docker-hub]/[nazwa-repo]
  ```
  np. `januszdev/cmentarna-polka`

- **Tag:**  
  ```
  ${BUILD_NUMBER}
  ```
  Dzięki temu każdy build dostanie unikalny tag 1, 2, 3...

- **Registry credentials → Add (Jenkins)**  
  - **Kind:** Username with Password  
  - **Username:** *użytkownik Docker Hub*  
  - **Password:** *hasło Docker Hub*  
  - **ID:** np. `dockerhub`

Wybierz utworzone credentials z listy.

Zapisz konfigurację.

---

# 6. Wywołanie builda (test)

Na serwerze GitLab, w katalogu repo wykonaj:

```bash
git add . && git commit -m "change" && git push -u origin master
```

Po pushu:

1. GitLab wywoła webhook → Jenkins.  
2. Jenkins pobierze repo.  
3. Zbuduje obraz z Dockerfile.  
4. Wypchnie go na Docker Hub z tagiem builda.

---

# 7. Weryfikacja na Docker Hub

Wejdź na:

```
https://hub.docker.com/
```

Przejdź do swojego repozytorium — powinieneś zobaczyć nowy obraz z numerem builda, np.:

- `1`
- `2`
- `3`

Każdy push = nowa wersja kontenera.

---

# Podsumowanie

W tym module:

- dodaliśmy Dockerfile + .dockerignore do projektu,
- zainstalowaliśmy plugin Docker Build and Publish,
- nadaliśmy Jenkinsowi dostęp do Dockera,
- skonfigurowaliśmy pipeline publikujący obraz do Docker Hub,
- potwierdziliśmy działanie wykonując testowy commit.

W kolejnym module zaczniemy **wdrażać kontenery na Docker Swarm** w sposób w pełni automatyczny.
