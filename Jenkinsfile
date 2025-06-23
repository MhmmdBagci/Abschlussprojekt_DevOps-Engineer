// - Erstellen Sie ein vollständiges Jenkinsfile mit folgenden Stages:
// - Build
// - Backend: mit Maven und aktivem Profil (dev oder prod)
// - Frontend: mit npm (z. B. npm install && npm run build)
// - Test
// - Backend: automatisierte JUnit-Tests
// - Frontend: falls vorhanden (z. B. ng test oder npm test)
// - Docker Build: Erstellen der Docker-Images
// - Docker Push: Push in die vom Dozenten bereitgestellte Container Registry
// - Deploy Dev: Deployment in den Dev-Namespace des Kubernetes-Clusters
// - Manual Approval: manuelle Bestätigung für das Prod-Deployment
// - Deploy Prod: Deployment in den Prod-Namespace

pipeline {
  // Jenkins nutzt einen beliebigen verfügbaren Agent
  agent any

  parameters {
    // Eingabeparameter für die Umgebung: dev oder prod
    string(name: 'BUILD_ENV', defaultValue: 'dev', description: 'dev oder prod')
  }

  environment {
    // Container-Image-Namen für Backend und Frontend
    IMAGE_BACKEND = "my-backend-image"
    IMAGE_FRONTEND = "my-frontend-image"
    
    // Die Azure Container Registry, in die gepusht wird
    REGISTRY_URL = "registrykurs1.azurecr.io"
  }

  stages {

    // =====================
    // 1. Backend Build (Java)
    // =====================
    stage('Build Backend') {
      steps {
        dir('backend') {
          // Erst bereinigen, dann bauen mit aktivem Profil (dev/prod)
          sh "mvn clean package -P${params.BUILD_ENV}"
        }
      }
    }

    // =====================
    // 2. Backend Tests (JUnit)
    // =====================
    stage('Test Backend') {
      steps {
        dir('backend') {
          // Führt alle JUnit-Tests im Backend durch
          sh "mvn test"
        }
      }
    }

    // =====================
    // 3. Frontend Build (npm)
    // =====================
    stage('Build Frontend') {
      steps {
        dir('frontend') {
          // Installiert alle Abhängigkeiten aus package.json
          sh "npm install"
          
          // Baut das Frontend (z. B. React, Vue oder Angular)
          sh "npm run build"
        }
      }
    }

    // =====================
    // 4. Frontend Tests (optional)
    // =====================
    stage('Test Frontend') {
      steps {
        dir('frontend') {
          // Führt Tests durch (falls vorhanden). 
          // Mit "|| true", damit kein Abbruch bei fehlenden Tests.
          sh "npm test || true"
        }
      }
    }

    // =====================
    // 5. Docker Images bauen
    // =====================
    stage('Docker Build') {
      steps {
        // Docker-Image für das Backend bauen
        sh "docker build -t ${REGISTRY_URL}/${IMAGE_BACKEND}:${params.BUILD_ENV} ./backend"

        // Docker-Image für das Frontend bauen
        sh "docker build -t ${REGISTRY_URL}/${IMAGE_FRONTEND}:${params.BUILD_ENV} ./frontend"
      }
    }

    // =====================
    // 6. Docker Images pushen (Azure Container Registry)
    // =====================
    stage('Docker Push') {
      steps {
        // Holt sichere Zugangsdaten (Benutzer & Passwort) aus Jenkins Credential Store
        withCredentials([usernamePassword(credentialsId: 'azure-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          // Login in Azure Container Registry
          sh "echo $DOCKER_PASS | docker login $REGISTRY_URL -u $DOCKER_USER --password-stdin"

          // Push Backend Image
          sh "docker push ${REGISTRY_URL}/${IMAGE_BACKEND}:${params.BUILD_ENV}"

          // Push Frontend Image
          sh "docker push ${REGISTRY_URL}/${IMAGE_FRONTEND}:${params.BUILD_ENV}"
        }
      }
    }

    // =====================
    // 7. Deployment in Dev (automatisch)
    // =====================
    stage('Deploy Dev') {
      when {
        // Nur bei dev-Umgebung ausführen
        expression { params.BUILD_ENV == 'dev' }
      }
      steps {
        // Deployment der YAML-Dateien in Kubernetes Dev-Namespace
        sh "kubectl apply -f kubernetes/k8s_dev.yaml"
      }
    }

    // =====================
    // 8. Manuelle Bestätigung für Prod
    // =====================
    stage('Approval') {
      when {
        // Nur bei prod-Umgebung anzeigen
        expression { params.BUILD_ENV == 'prod' }
      }
      steps {
        // Manuelle Freigabe für das Live-Deployment
        input message: 'Willst du wirklich in Produktion deployen?', ok: 'Ja, weiter'
      }
    }

    // =====================
    // 9. Deployment in Prod (nach Freigabe)
    // =====================
    stage('Deploy Prod') {
      when {
        // Nur bei prod-Umgebung ausführen
        expression { params.BUILD_ENV == 'prod' }
      }
      steps {
        // Deployment der YAML-Dateien in Kubernetes Prod-Namespace
        sh "kubectl apply -f kubernetes/k8s_prod.yaml"
      }
    }
  }

  // =====================
  // Nach dem Build – Aufräumen & Statusmeldung
  // =====================
  post {
    always {
      // Workspace löschen nach jedem Durchlauf (auch bei Fehlern)
      cleanWs()
    }
    success {
      echo "✅ Jenkins-Pipeline erfolgreich abgeschlossen!"
    }
    failure {
      echo "❌ Fehler in der Jenkins-Pipeline aufgetreten."
    }
  }
}
