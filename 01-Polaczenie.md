# Docker CI/CD — Moduł 01  
## Połączenie z labem

W tym pierwszym module przygotujemy środowisko pracy, łącząc się z maszynami laboratorium:  
- **Jenkins** – serwer CI, który będzie wykonywał buildy, testy i pipeline’y,  
- **GitLab** – repozytorium kodu oraz system zarządzania projektami,  
- **manager01** – manager klastra Docker Swarm,  
- **worker01** – węzeł roboczy klastra Swarm, na którym będą uruchamiane kontenery.

Zanim rozpoczniemy pracę z CI/CD, musimy uzyskać dostęp do wszystkich serwerów.

---

# Pobranie klucza dostępowego

Lab udostępnia nam klucz prywatny pozwalający na logowanie do wszystkich maszyn.  
Pobierz go komendą:

```bash
wget https://raw.githubusercontent.com/inleo-pl/Warsztaty-Docker-CI-CD/master/Docker_CI_CD.pem
```

Nadaj mu odpowiednie uprawnienia (SSH odrzuci klucz o zbyt „szerokich” prawach):

```bash
chmod 600 /miejsce/gdzie/jest/Docker_CI_CD.pem
```

---

# Logowanie do poszczególnych maszyn

Teraz możemy zalogować się do każdej maszyny.  
Poniżej polecenia — podmień `/miejsce/gdzie/jest/` na właściwą ścieżkę do klucza.

---

## Jenkins

```bash
ssh -i /miejsce/gdzie/jest/Docker_CI_CD.pem ubuntu@jankins
```

Ta maszyna będzie hostować nasz serwer **Jenkins**, który wykona pipeline CI/CD.

---

## GitLab

```bash
ssh -i /miejsce/gdzie/jest/Docker_CI_CD.pem ubuntu@gitlab
```

Tutaj znajduje się nasz **GitLab**, w którym będziemy trzymać repozytorium aplikacji, runner’y i definicje pipeline’ów.

---

## Manager Docker Swarm

```bash
ssh -i /miejsce/gdzie/jest/Docker_CI_CD.pem ubuntu@manager01
```

Na tym węźle:

- uruchomimy Docker Swarm,
- będziemy wdrażać serwisy aplikacyjne,
- Jenkins będzie tutaj wypychał kontenery.

---

## Worker Docker Swarm

```bash
ssh -i /miejsce/gdzie/jest/Docker_CI_CD.pem ubuntu@worker01
```

To węzeł roboczy (worker) – będzie otrzymywał workloady ze Swarma.

---

# Podsumowanie

W tym module:

- pobrałeś klucz dostępu,
- przygotowałeś uprawnienia SSH,
- nawiązałeś połączenia z czterema maszynami labowymi.

To oznacza, że środowisko jest gotowe do pracy!  
W kolejnym module rozpoczniemy instalację i konfigurację **GitLab + Jenkins + Docker Swarm**, aby zbudować kompletny pipeline CI/CD.
