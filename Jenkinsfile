pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  environment {
    IMAGE_NAME = "react-hello"
    COMPOSE_FILE = "rayando-architecture/docker-compose.yml"
  }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Install & Test') {
      // Corre npm dentro de un contenedor Node
      agent { docker { image 'node:20-alpine' } }
      steps {
        sh '''
          node -v
          npm -v
          npm ci
          npm run test --if-present
        '''
      }
    }

    stage('Build Image') {
      steps {
        sh '''
          COMMIT=$(git rev-parse --short HEAD)
          docker build --pull -t ${IMAGE_NAME}:$COMMIT -t ${IMAGE_NAME}:latest .
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          docker compose -f ${COMPOSE_FILE} up -d --no-deps --force-recreate reactapp
          docker image prune -f
        '''
      }
    }
  }
}
