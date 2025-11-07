pipeline {
  agent any

  options {
    disableConcurrentBuilds()
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '15'))
  }

  environment {
    // ---- à adapter si besoin ----
    NAMESPACE     = 'daliii'
    IMAGE_SERVER  = "${NAMESPACE}/mern-server"
    IMAGE_CLIENT  = "${NAMESPACE}/mern-client"
    DOCKER_CREDS  = 'dockerhub' // CredentialsID Jenkins (username/password Docker Hub)
    COMMIT        = ''          // sera rempli au stage Checkout
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
        script {
          COMMIT = sh(script: "git rev-parse --short=8 HEAD", returnStdout: true).trim()
          env.COMMIT = COMMIT
          echo "Commit: ${COMMIT}"
        }
      }
    }

    stage('Docker login') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}",
                                          usernameVariable: 'DH_USER',
                                          passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
          '''
        }
      }
    }

    stage('Build & Push SERVER') {
      when {
        changeset pattern: 'server/**', comparator: 'INCLUDE'
      }
      steps {
        sh '''
          echo "Building ${IMAGE_SERVER}:${COMMIT}"
          docker build -t ${IMAGE_SERVER}:${COMMIT} server
          docker tag  ${IMAGE_SERVER}:${COMMIT} ${IMAGE_SERVER}:latest
          docker push ${IMAGE_SERVER}:${COMMIT}
          docker push ${IMAGE_SERVER}:latest
        '''
      }
    }

    stage('Build & Push CLIENT') {
      when {
        changeset pattern: 'client/**', comparator: 'INCLUDE'
      }
      steps {
        sh '''
          echo "Building ${IMAGE_CLIENT}:${COMMIT}"
          docker build -t ${IMAGE_CLIENT}:${COMMIT} client
          docker tag  ${IMAGE_CLIENT}:${COMMIT} ${IMAGE_CLIENT}:latest
          docker push ${IMAGE_CLIENT}:${COMMIT}
          docker push ${IMAGE_CLIENT}:latest
        '''
      }
    }
  }

  post {
    always {
      sh 'docker system prune -af || true'
    }
    success {
      echo "✅ Images poussées: ${IMAGE_SERVER}:${COMMIT} / latest, ${IMAGE_CLIENT}:${COMMIT} / latest"
    }
    failure {
      echo "❌ Build échoué — vérifier la console Jenkins."
    }
  }
}
