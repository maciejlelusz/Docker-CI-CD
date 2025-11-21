# Docker CI/CD — Moduł 09  
## Docker Bench Security + AppArmor — audyt i zabezpieczanie środowiska

W tym module zajmiemy się **bezpieczeństwem środowiska kontenerowego**, co jest naturalnym etapem po zbudowaniu działającego pipeline’u CI/CD.  
Skupimy się na:

- audycie serwera Docker za pomocą **Docker Bench Security**,  
- użyciu praktycznych poleceń do inspekcji kontenerów,  
- konfiguracji **AppArmor** — jednego z najważniejszych mechanizmów bezpieczeństwa Linux.

To ważne, aby CI/CD nie tylko wdrażało aplikacje, ale też zapewniało ich bezpieczeństwo.

---

# 1. Docker Bench Security — audyt bezpieczeństwa Dockera

**Docker Bench Security** to oficjalny skrypt od Docker, który wykonuje zestaw testów bezpieczeństwa zgodnych z wytycznymi CIS (Center for Internet Security).

## Instalacja i uruchomienie

Na dowolnym serwerze Docker (np. manager01 lub jenkins):

```bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sh docker-bench-security.sh
```

Skrypt wykona:

- analizę konfiguracji Dockera,
- sprawdzenie uprawnień użytkowników,
- weryfikację konfiguracji demonów,
- kontrole bezpieczeństwa kernelowych mechanizmów (AppArmor, Seccomp, namespaces, cgroups),
- analizę konfiguracji sieci kontenerowej.

Na końcu wygeneruje szczegółowy raport z oceną poszczególnych kategorii.

---

# 2. Przydatne komendy audytowe

Poniższe polecenia pomagają sprawdzić kluczowe aspekty bezpieczeństwa kontenerów.

---

## Lista kontenerów wraz z mapowaniami portów

```bash
docker ps --quiet | xargs docker inspect --format '{{ .Id }}: Ports={{ .NetworkSettings.Ports }}'
```

Informacje, które otrzymasz:

- ID kontenera,
- aktualne mapowania portów (host → container).

Przydatne do szybkiego wykrywania niepotrzebnie otwartych portów.

---

## Propagacja mountów (osadzanie systemu plików)

```bash
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Propagation={{range $mnt := .Mounts}} {{json $mnt.Propagation}} {{end}}'
```

To polecenie pokazuje, jak kontener współdzieli przestrzeń montowania:

- `rprivate` → najbezpieczniejsze (brak propagacji),
- `shared` → potencjalnie ryzykowne,
- `slave` → częściowa propagacja.

Jest to krytyczny aspekt bezpieczeństwa kontenerów (mount propagation może prowadzić do eskalacji uprawnień).

---

# 3. AppArmor — dodatkowa warstwa bezpieczeństwa kontenerów

**AppArmor** to system kontroli dostępu oparty na profilach, który ogranicza, co proces w kontenerze może zrobić.

Docker domyślnie wspiera AppArmor — możemy tworzyć własne profile, aby zwiększyć odporność aplikacji.

---

## 3.1. Przygotowanie katalogu na profile AppArmor

```bash
mkdir -p /etc/apparmor.d/containers/
```

---

## 3.2. Tworzenie profilu bezpieczeństwa dla Nginx

Plik:

```bash
nano /etc/apparmor.d/containers/docker-nginx
```

Treść profilu pobierz z oficjalnej dokumentacji Dockera:

```
https://docs.docker.com/engine/security/apparmor/#nginx-example-profile
```

Profil ten:

- ogranicza dostęp do sieci,
- blokuje zapisy poza określonymi katalogami,
- ustala dozwolone syscalle i operacje,
- zapewnia izolację procesów Nginxa.

---

## 3.3. Załadowanie profilu AppArmor

Po przygotowaniu pliku:

```bash
apparmor_parser -r -W /etc/apparmor.d/containers/docker-nginx
```

Flagi:

- `-r` — przeładuj profil,
- `-W` — pokaż ostrzeżenia (np. brakujące katalogi).

Po załadowaniu można uruchamiać kontenery z użyciem tego profilu:

```bash
docker run --security-opt apparmor=docker-nginx nginx
```

---

# Podsumowanie

W tym module nauczyłeś się:

- jak wykonać audyt bezpieczeństwa środowiska kontenerowego,
- jak analizować konfigurację Dockera i kontenerów,
- jak korzystać z AppArmor, aby zwiększyć bezpieczeństwo produkcyjnych aplikacji,
- jak istotne są mechanizmy hardeningu w kontekście CI/CD oraz Docker Swarm.

To zamyka część szkolenia dotyczącą CI/CD — masz pipeline, klaster Swarm i podstawy bezpieczeństwa produkcyjnego.
