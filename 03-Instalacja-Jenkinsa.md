# Docker CI/CD — Moduł 03  
## Instalacja i uruchomienie Jenkins

W tym module zajmiemy się instalacją **Jenkinsa** – głównego narzędzia CI/CD, które będzie wykonywać pipeline’y, testy, buildy i wdrożenia.  
Pokażemy dwie metody instalacji:

1. **Metoda tradycyjna (instalacja na systemie hosta)**
2. **Instalacja jako kontener Dockera**

---

# 1. Instalacja tradycyjna (systemowa)

Ta metoda instaluje Jenkinsa jako usługę systemową (systemd), dzięki czemu działa stabilnie nawet po restarcie maszyny.

### Krok 1 — instalacja JRE

Jenkins wymaga środowiska Java:

```bash
sudo apt-get install default-jre
```

---

### Krok 2 — dodanie repozytorium Jenkins

Pobieramy klucz GPG:

```bash
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
```

Dodajemy repozytorium:

```bash
sudo echo deb https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
```

Aktualizujemy repozytoria:

```bash
sudo apt-get -y update
```

---

### Krok 3 — instalacja Jenkins

```bash
sudo apt-get -y install jenkins
```

Po instalacji uruchamiamy usługę:

```bash
sudo systemctl start jenkins
```

Sprawdzamy status:

```bash
sudo systemctl status jenkins
```

Powinno wyświetlić `active (running)`.

---

# 2. Logowanie do Jenkins

Po uruchomieniu Jenkins generuje jednorazowe hasło administratora.

Odczytaj je:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Następnie otwórz przeglądarkę i przejdź do:

```
http://jenkins:8080
```

Zamień `jenkins` na adres swojej maszyny (IP lub hostname).  
Po wejściu:

- wprowadź hasło początkowe,
- przejdź przez kreator instalacji,
- zainstaluj zalecane pluginy.

---

# 3. Instalacja Jenkinsa jako kontener Docker

Jeśli wolisz uproszczoną instalację, Jenkins może działać jako kontener:

```bash
sudo docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
```

Objaśnienie:

- port **8080** — panel webowy,
- port **50000** — komunikacja z agentami Jenkins (np. build workers),
- obraz `jenkins/jenkins:lts` — stabilna wersja Long Term Support.

Po uruchomieniu kontener zapisze dane w wewnętrznym systemie plików — w środowisku produkcyjnym warto dodać wolumeny.

Przykład z wolumenami:

```bash
sudo docker run -d \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

---

# Podsumowanie

W tym module:

- zainstalowaliśmy Jenkins w sposób tradycyjny lub kontenerowy,
- uzyskaliśmy dostęp do interfejsu webowego,
- przygotowaliśmy środowisko do dalszej konfiguracji CI/CD.

W kolejnym kroku będziemy konfigurować **GitLab**, **GitLab Runner** oraz integrację z Jenkins i Docker Swarm, aby pipeline’y mogły budować i wdrażać aplikacje automatycznie.
