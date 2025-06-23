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
          sh "npm install"
          sh "npm run build"
        }
      }
    }

    // =====================
    // 4. Frontend Tests
    // =====================
    stage('Test Frontend') {
      steps {
        dir('frontend') {
          sh "npm test || true"
        }
      }
    }

    // =====================
    // 5. Docker Build
    // =====================
    stage('Docker Build') {
      steps {
        sh "docker build -t ${REGISTRY_URL}/${IMAGE_BACKEND}:${params.BUILD_ENV} ./backend"
        sh "docker build -t ${REGISTRY_URL}/${IMAGE_FRONTEND}:${params.BUILD_ENV} ./frontend"
      }
    }

    // =====================
    // 6. Docker Push
    // =====================
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'azure-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh "echo $DOCKER_PASS | docker login $REGISTRY_URL -u $DOCKER_USER --password-stdin"
          sh "docker push ${REGISTRY_URL}/${IMAGE_BACKEND}:${params.BUILD_ENV}"
          sh "docker push ${REGISTRY_URL}/${IMAGE_FRONTEND}:${params.BUILD_ENV}"
        }
      }
    }

    // =====================
    // 7. Deployment Dev
    // =====================
    stage('Deploy Dev') {
      when {
        expression { params.BUILD_ENV == 'dev' }
      }
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          withCredentials([file(credentialsId: 'kubeconfig-dev', variable: 'KUBECONFIG')]) {
            sh "kubectl apply -f kubernetes/k8s_dev.yaml"
          }
        }
      }
    }

    // =====================
    // 8. Deployment Prod
    // =====================
    stage('Deploy Prod') {
      when {
        expression { params.BUILD_ENV == 'prod' }
      }
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')]) {
            sh "kubectl apply -f kubernetes/k8s_prod.yaml"
          }
        }
      }
    }

  } // ENDE stages

  // =====================
  // Aufräumen & Status
  // =====================
  post {
    always {
      sh 'rm -rf ./*'
    }
    success {
      echo "✅ Jenkins-Pipeline erfolgreich abgeschlossen!"
    }
    failure {
      echo "❌ Fehler in der Jenkins-Pipeline aufgetreten."
    }
  }
} // ENDE pipeline
