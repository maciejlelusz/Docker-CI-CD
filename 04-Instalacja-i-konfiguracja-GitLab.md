# Docker CI/CD — Moduł 04  
## Instalacja GitLab EE + konfiguracja użytkownika dla Jenkinsa

W tym module:

- pobierzemy **licencję trial GitLab EE** (wymaganą, ponieważ integracja z Jenkins CI działa tylko w wersji GitLab Enterprise Edition),
- zainstalujemy GitLab EE na dedykowanym serwerze,
- skonfigurujemy użytkownika serwisowego “jenkins”,
- wygenerujemy token dostępu (Impersonation Token),
- stworzymy pierwsze repozytorium i wypchniemy do niego kod.

---

# 1. Pobranie licencji trial GitLab

GitLab EE wymaga licencji. Pobierzemy wersję trial (30 dni):

```
https://about.gitlab.com/free-trial/
```

Plik licencji będzie potrzebny podczas konfiguracji GitLab UI.

---

# 2. Instalacja GitLab EE na serwerze `gitlab`

Zaloguj się na maszynę GitLab, a następnie wykonaj poniższe polecenia.

### Krok 1 — aktualizacja systemu

```bash
sudo apt-get update
sudo apt-get -y upgrade
```

### Krok 2 — zależności

```bash
sudo apt-get install -y curl openssh-server ca-certificates
sudo apt-get install -y postfix
```

`postfix` obsługuje wysyłkę maili, np. powiadomień o pipeline’ach.

---

### Krok 3 — dodanie repozytorium GitLab EE

```bash
sudo curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```

---

### Krok 4 — instalacja GitLab EE

Ustaw własny adres URL (hostname lub IP):

```bash
sudo EXTERNAL_URL="http://[nazwahosta]" apt-get install gitlab-ee
```

Po zakończeniu instalacji GitLab będzie dostępny w sieci.

---

# 3. Dostęp do GitLab UI

Otwórz przeglądarkę i przejdź pod adres:

```
http://gitlab
```

Pierwsze uruchomienie poprosi o ustawienie hasła administratora (`root`).

---

# 4. Wgranie licencji GitLab EE

W GitLab UI:

```
Admin Area → License → Upload New License
```

Wgraj plik pobrany ze strony triala.

Po aktywacji licencji będziesz mógł korzystać z funkcji integracji z Jenkins CI.

---

# 5. Konfiguracja użytkownika „jenkins”

Jenkins będzie logował się do GitLab przez API.  
Najpierw stworzymy użytkownika systemowego GitLab:

1. Wejdź w:
   ```
   Admin Area
     → Users
     → New user
   ```

2. Ustaw pola:
   - **Name:** `jenkins`
   - **Access level:** Admin (dla pełnej integracji)

3. Zapisz użytkownika.

---

# 6. Generowanie Impersonation Token (API Token)

Przechodzimy do zakładki użytkownika `jenkins`:

```
Users → jenkins → Impersonation Tokens
```

Tworzymy nowy token:

- **Name:** `jenkins`
- **Scopes:** `api`

Po wygenerowaniu:

- **skopiuj token** (nie będzie widoczny ponownie),
- kliknij **Edit** w prawym górnym rogu profilu,
- ustaw hasło użytkownika (ważne!),
- zapisz zmiany.

Token będzie używany przez Jenkinsa do komunikacji z GitLabem.

---

# 7. Stworzenie nowego projektu GitLab  
Repozytorium, na którym będziemy ćwiczyć CI/CD.

Kliknij **+ (plus)** w prawym górnym rogu, następnie:

```
New project → Project name: Cmentarna-Polka
```

Utwórz projekt.

---

# 8. Klonowanie repozytorium i pierwszy commit

Na serwerze GitLab (lub na dowolnej innej maszynie z dostępem):

### Sprawdź wersję Git:

```bash
git --version
```

### Sklonuj repozytorium:

```bash
git clone http://gitlab/root/Cmentarna-Polka.git
```

### Dodaj pierwszy plik:

```bash
cd Cmentarna-Polka
touch README.md
git add README.md
git commit -m "add README"
```

### Wypchnięcie zmian:

```bash
git push -u origin master
```

Repozytorium jest gotowe — w kolejnych modułach zintegrujemy je z Jenkins i zbudujemy prawdziwy pipeline CI/CD!

---

# Podsumowanie

W tym module:

- zainstalowałeś GitLab EE,
- aktywowałeś licencję,
- utworzyłeś użytkownika dla Jenkinsa,
- wygenerowałeś token API,
- stworzyłeś pierwsze repozytorium i umieściłeś w nim kod.

Następny krok to integracja GitLab → Jenkins → Docker Swarm, aby możliwe było pełne CI/CD.
