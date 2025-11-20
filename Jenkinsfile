pipeline {
  agent {
    docker {
      image 'node:18-bullseye'
      args '-u root:root'
    }
  }

  environment {
    NODE_ENV = 'test'
    DOCKER_IMAGE = "${env.JOB_NAME}:${env.BUILD_NUMBER}"
  }

  options {
    timestamps()
    ansiColor('xterm')
    skipDefaultCheckout(false)
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test -- --runInBand || true'
      }
    }

    stage('Build Docker Image') {
      when {
        expression { return fileExists('Dockerfile') }
      }
      steps {
        sh 'docker --version || true'
        sh 'docker build -t "${DOCKER_IMAGE}" .'
      }
    }

    stage('Push Docker Image') {
      when {
        allOf {
          expression { return fileExists('Dockerfile') }
          expression { return env.DOCKER_REGISTRY_HOST && env.DOCKER_REGISTRY_CREDENTIALS }
        }
      }
      steps {
        withCredentials([usernamePassword(credentialsId: "${env.DOCKER_REGISTRY_CREDENTIALS}", usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
          sh '''
            echo "$REG_PASS" | docker login -u "$REG_USER" --password-stdin ${DOCKER_REGISTRY_HOST}
            docker tag "${DOCKER_IMAGE}" ${DOCKER_REGISTRY_HOST}/${DOCKER_IMAGE}
            docker push ${DOCKER_REGISTRY_HOST}/${DOCKER_IMAGE}
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'coverage/**,reports/**,npm-debug.log', allowEmptyArchive: true
      cleanWs()
    }
    success {
      echo 'Pipeline succeeded.'
    }
    failure {
      echo 'Pipeline failed.'
    }
  }
}
