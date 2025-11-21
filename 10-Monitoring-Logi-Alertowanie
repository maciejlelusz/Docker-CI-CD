# Docker CI/CD — Moduł 10
## Monitoring, logi i alertowanie w środowisku Docker Swarm + Jenkins + GitLab

Po zbudowaniu kompletnego pipeline’u CI/CD, wdrożeniu aplikacji na Docker Swarm oraz konfiguracji podstawowych mechanizmów bezpieczeństwa, czas na ostatni fundament środowiska produkcyjnego: **monitoring, logowanie i alerty**.

W tym module:

- Zainstalujemy i uruchomimy **Prometheus + cAdvisor** do monitoringu kontenerów  
- Dodamy **Grafana** do wizualizacji metryk  
- Skonfigurujemy **Docker log drivers** i zbieranie logów  
- Pokażemy integrację alertów (Slack / e-mail)  
- Omówimy dobre praktyki monitoringu dla CI/CD

To sprawi, że Twoje środowisko stanie się **przejrzyste, monitorowane i gotowe do reagowania na awarie**.

---

# 1. Monitoring zasobów — Prometheus + cAdvisor

Monitoring kontenerów można wdrożyć na kilka sposobów, ale najpopularniejszym i najskuteczniejszym zestawem jest:

- **Prometheus** — baza metryk + silnik zapytań
- **cAdvisor** — zbiera metryki kontenerów Docker (CPU, RAM, IO, Restart Count)
- **Grafana** — wizualizacja i panele monitoringu

## 1.1. Uruchomienie cAdvisor na każdym węźle Swarm

Na `manager01` i `worker01`:

```bash
sudo docker service create \
  --name=cadvisor \
  --mode=global \
  --publish mode=host,target=8080,published=8080 \
  --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  --mount type=bind,src=/,dst=/rootfs,ro \
  --mount type=bind,src=/sys,dst=/sys,ro \
  --mount type=bind,src=/var/lib/docker,dst=/var/lib/docker,ro \
  gcr.io/cadvisor/cadvisor:latest
```

Now każdy węzeł wystawia statystyki pod adresem:

```
http://[IP-węzła]:8080
```

---

## 1.2. Uruchomienie Prometheusa

Stwórz plik `prometheus.yml`:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: cadvisor
    static_configs:
      - targets:
        - 'manager01:8080'
        - 'worker01:8080'
```

Uruchom Prometheus:

```bash
sudo docker service create \
  --name=prometheus \
  --publish 9090:9090 \
  --mount type=bind,src=$(pwd)/prometheus.yml,dst=/etc/prometheus/prometheus.yml \
  prom/prometheus
```

Prometheus będzie dostępny pod:

```
http://manager01:9090
```

---

# 2. Wizualizacja — Grafana

Uruchom Grafanę na Swarm:

```bash
sudo docker service create \
  --name=grafana \
  --publish 3000:3000 \
  grafana/grafana:latest
```

Grafana dostępna pod:

```
http://manager01:3000
```

Domyślne dane logowania:

- user: `admin`
- pass: `admin`

Następnie:

1. Dodaj **Prometheus** jako źródło danych (`http://prometheus:9090`)
2. Importuj dashboardy:
   - **Docker / Swarm Monitoring**
   - **Container Resource Usage**
   - **Host Metrics**

Są dostępne za darmo na https://grafana.com/dashboards.

---

# 3. Logowanie — Docker Log Drivers + ELK (opcjonalnie)

Docker domyślnie używa log drivera **json-file**. Można go sprawdzić:

```bash
docker inspect <ID-kontenera> --format '{{ .HostConfig.LogConfig }}'
```

Aby zbierać logi globalnie, masz dwie opcje:

---

## 3.1. Logowanie do syslog

Edytuj `/etc/docker/daemon.json` (na każdym węźle):

```json
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "udp://manager01:514"
  }
}
```

Restart:

```bash
sudo systemctl restart docker
```

---

## 3.2. Stack ELK (Elasticsearch + Logstash + Kibana)

Najpopularniejsze rozwiązanie do logowania kontenerów.

Przykład uruchomienia:

```bash
docker service create --name kibana --publish 5601:5601 kibana:latest
docker service create --name elasticsearch --publish 9200:9200 elasticsearch:7
docker service create --name logstash logstash:latest
```

Docker → Logstash → Elasticsearch → Kibana.

Możesz monitorować wszystkie logi kontenerów z całego Swarm.

---

# 4. Alertowanie — Slack / Email

Prometheus obsługuje alerty poprzez **Alertmanager**.

Przykładowa instalacja:

```bash
docker service create \
  --name=alertmanager \
  --publish 9093:9093 \
  prom/alertmanager
```

Konfiguracja `alertmanager.yml`:

### Slack:

```yaml
receivers:
- name: slack-alerts
  slack_configs:
  - channel: "#devops"
    send_resolved: true
    api_url: https://hooks.slack.com/services/XXXXX/YYYYY/ZZZZZ
```

### Email:

```yaml
email_configs:
  - to: "admin@example.com"
    from: "ci@example.com"
    smarthost: "smtp.example.com:587"
    auth_username: "ci@example.com"
    auth_password: "SECRET"
```

---

# 5. Co monitorować w CI/CD?

Najważniejsze metryki dla produkcyjnego CI/CD:

## Jenkins:
- Load Average
- Czas wykonania jobów
- Kolejka buildów
- Status workerów
- Dostępność portu 8080

## Docker Swarm:
- CPU/RAM kontenerów
- Restart Count
- Czas życia kontenerów
- OBB (Out-of-memory kills)
- Status usług (Running/Down/Preparing)
- Skalowanie usług

## GitLab:
- Health checks `/-/health`
- API latency
- Size repozytoriów
- Webhook delivery status

## Host system:
- CPU/memory/disk
- IOPS / IO latency
- Sieć (RX/TX)
- Użycie katalogu `/var/lib/docker`

---

# Podsumowanie

W tym module wdrożyliśmy kompletny monitoring i logowanie środowiska CI/CD:

- Prometheus + cAdvisor → zbieranie metryk Docker Swarm  
- Grafana → wizualizacja i dashboardy  
- Syslog/ELK → centralizacja logów  
- Alertmanager → powiadomienia (Slack/e-mail)  

Masz teraz **pełny stack DevOps**:

> GitLab → Jenkins → Docker Swarm → Monitoring → Alerting → Continuous Delivery
