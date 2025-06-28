# Abschlussprojekt DevOps Engineer

## Projektziel

Ziel des Projekts war es, eine vollständige CI/CD-Pipeline für eine Beispielanwendung zu erstellen. Die Anwendung besteht aus einem Java-Backend (Spring Boot) und einem JavaScript-Frontend. Die Deployments sollten automatisch über Jenkins in ein Kubernetes-Cluster erfolgen – getrennt nach Entwicklungs- und Produktionsumgebung.

## Inhalte des Projekts

- Aufbau eines Git-Repositories mit sauberer Projektstruktur
- Erstellung eines Jenkinsfile mit allen notwendigen Schritten:
  - Build und Test von Backend (Maven)
  - Build und Test von Frontend (npm)
  - Docker Build & Push
  - Kubernetes-Deployment (dev automatisch, prod mit manueller Freigabe)
- Nutzung von Maven-Profilen (`dev` und `prod`) zur Konfigurationssteuerung
- Nutzung von zwei Kubernetes-Namespaces: `dev` und `prod`
- Bereitstellung des Frontends über LoadBalancer-Service

## Umgebungen

### Dev
- Port Backend: 8081
- Deployment erfolgt direkt nach dem Push
- Logs im Dev-Modus detaillierter

### Prod
- Port Backend: 8082
- Deployment erfolgt nur nach manueller Freigabe
- Logs reduziert auf Fehlerausgabe

## Kubernetes

Alle Deployments und Services sind in einer Datei je Umgebung enthalten (`dev.yaml` / `prod.yaml`).  
Die Deployments enthalten Umgebungsvariablen zur Steuerung des aktiven Spring-Profils.

## CI/CD mit Jenkins

Der gesamte Ablauf ist über Jenkins automatisiert:

1. Code wird aus dem Git-Repository geladen
2. Backend und Frontend werden gebaut und getestet
3. Docker-Images werden erstellt und in die Registry hochgeladen
4. Deployment in `dev`
5. Manuelle Freigabe für `prod` (optional)

## Präsentation

Gezeigt werden:
- Live-Durchlauf der Jenkins-Pipeline
- Übersicht über die Kubernetes-Ressourcen (`kubectl get pods` / `svc`)
- Screenshots der einzelnen Schritte

---

**Autor:** Muhammed Bagci



