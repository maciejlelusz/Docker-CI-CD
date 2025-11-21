# Docker CI/CD — Moduł 05  
## Integracja GitLab → Jenkins (webhooks + tokeny + konfiguracja projektów)

W tym module połączymy **GitLab** i **Jenkins**, aby każdy push do repozytorium automatycznie wywoływał pipeline CI w Jenkinsie.

Zrobimy to w trzech krokach:

1. Pobierzemy token z GitLab i zainstalujemy wtyczkę GitLab w Jenkinsie  
2. Skonfigurujemy połączenie Jenkins → GitLab  
3. Skonfigurujemy integrację GitLab → Jenkins (webhook)  
4. Zweryfikujemy działanie dodając plik do repo

---

# 1. Pobranie tokenu z GitLab (dla Jenkinsa)

Token API jest wymagany, aby Jenkins mógł komunikować się z GitLabem.

W GitLab UI przejdź do:

```
Admin Area → Users → jenkins → Impersonation Tokens
```

Następnie:

- utwórz token,
- ustaw scope: **api**,
- **skopiuj token** – będzie potrzebny w Jenkinsie.

---

# 2. Instalacja wtyczki GitLab w Jenkins

Wejdź do panelu Jenkinsa:

```
http://jenkins:8080
```

Przejdź do:

```
Zarządzaj Jenkinsem → Zarządzaj wtyczkami → Dostępne
```

Znajdź wtyczkę:

- **GitLab**

Zainstaluj:

```
→ Zainstaluj bez restartu
→ Uruchom ponownie gdy instalacja zostanie zakończona
```

Po restarcie wtyczka jest gotowa.

---

# 3. Konfiguracja połączenia Jenkins → GitLab

Przejdź do:

```
Zarządzaj Jenkinsem → Skonfiguruj system → GitLab
```

Ustaw:

- **Connection Name:** `GitLab`
- **GitLab Host URL:** `http://gitlab`

### Dodanie poświadczeń (token API)

Kliknij:

```
Credentials → Add
```

Ustaw parametry:

- **Domain:** Global
- **Kind:** GitLab API token  
- **Scope:** Global
- **API token:** (token pobrany z GitLab)
- **ID:** `jenkins`

Zapisz, następnie kliknij:

```
Test Connection
```

Jeśli działa — Jenkins połączył się z GitLabem poprawnie.

---

# 4. Dodanie projektu w Jenkins

Tworzymy nowy projekt:

```
Nowy projekt → Nazwa: Cmentarna-Polka-Deploy
```

Wybierz typ:

```
→ Ogólny projekt
```

### Konfiguracja repozytorium

Sekcja:

```
Repozytorium kodu → Git
```

Ustaw:

- **Repository URL:**  
  ```
  http://gitlab/root/Cmentarna-Polka.git
  ```

### Dodanie poświadczeń GitLab (username/password)

W sekcji Credentials kliknij:

```
Add → Jenkins
```

Ustaw:

- **Username:** `jenkins`
- **Password:** *hasło użytkownika jenkins na GitLab*
- **ID:** `jenkins`

Zapisz i wybierz poświadczenie `jenkins` z listy.

### Włączenie triggera GitLab

Zaznacz:

```
Build when a change is pushed to GitLab.
GitLab CI Service URL: http://jenkins:8080/project/Cmentarna-Polka-Deploy
```

To adres webhooka, który GitLab będzie wywoływać po każdym pushu.

Kliknij **Zapisz**.

---

# 5. Dodanie użytkownika dla GitLab (w Jenkins)

Jenkins wymaga lokalnego użytkownika, który będzie używany przez GitLab CI Service:

```
Zarządzaj Jenkinsem → Zarządzaj użytkownikami → Stwórz użytkownika
```

Ustaw:

- **login:** `gitlab`
- hasło dowolne (zapamiętaj!)

---

# 6. Konfiguracja GitLab → Jenkins (webhook)

Najpierw włączamy możliwość wykonywania zapytań do lokalnej sieci (Jenkins zwykle jest w tej samej sieci co GitLab).

Przejdź do GitLab Admin Area:

```
Admin Area → Settings → Outbound requests → Expand
```

Zaznacz:

- **Allow requests to the local network from hooks and services**

Kliknij **Save Changes**.

---

# 7. Włączenie Jenkins CI Service w projekcie GitLab

Przejdź do repozytorium:

```
Cmentarna-Polka → Settings → Integration → Project services
```

Znajdź usługę:

```
Jenkins CI
```

Kliknij aby otworzyć i ustaw:

- **Active:** ✔
- **Jenkins URL:**  
  ```
  http://[jenkins]:8080/
  ```
- **Project name:** `Cmentarna-Polka-Deploy`
- **Login:** użytkownik `gitlab` utworzony w Jenkins

Zapisz ustawienia.

---

# 8. Test działania integracji

Na serwerze GitLab uruchom:

```bash
touch index.html
git add index.html
git commit -m "add index.html"
git push -u origin master
```

Po pushu:

- GitLab wywoła webhook → Jenkins,
- Jenkins uruchomi projekt **Cmentarna-Polka-Deploy**,
- w Jenkins UI pojawi się nowe zadanie Build #1.

Integracja działa!

---

# Podsumowanie

W tym module:

- pobraliśmy token API GitLab,
- zainstalowaliśmy i skonfigurowaliśmy wtyczkę GitLab w Jenkinsie,
- skonfigurowaliśmy dwukierunkową komunikację (Jenkins → GitLab i GitLab → Jenkins),
- dodaliśmy projekt CI,
- przetestowaliśmy trigger push-to-build.

Pipeline jest gotowy — w kolejnym module zaczniemy budować obrazy Docker i wdrażać je na Swarm!
