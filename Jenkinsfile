pipeline {
  agent { label 'CYBR-3120-Appserver' }   // run everything on this node
  environment {
    DOCKERHUB_CREDENTIALS = 'cybr-3120'
    IMAGE_NAME = 'stajose/chatapp'
  }
  options { skipDefaultCheckout(true) }    // avoid implicit checkout on controller

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('BUILD-AND-TAG') {
      steps {
        script {
          echo "Building Docker image ${IMAGE_NAME}..."
          def app = docker.build("${IMAGE_NAME}")
          app.tag('latest', true)
        }
      }
    }

    stage('POST-TO-DOCKERHUB') {
      steps {
        script {
          echo "Pushing image ${IMAGE_NAME}:latest to Docker Hub"
          docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
            docker.image("${IMAGE_NAME}:latest").push()
          }
        }
      }
    }

    stage('SECURITY-IMAGE-SCANNER') {
      steps { sh 'echo Scanning Docker image for vulnerabilities...' }
    }

    stage('Pull-image-server') {
      steps { sh 'echo Pulling image on server...' }
    }

    stage('DAST') {
      steps { sh 'echo Running DAST scan...' }
    }

    stage('DEPLOYMENT') {
      steps {
        echo 'Starting deployment using docker compose...'
        sh '''
          set -e
          docker compose down || true
          docker compose pull || true
          docker compose up -d
          docker ps
        '''
        echo 'Deployment completed successfully'
      }
    }
  }
}
