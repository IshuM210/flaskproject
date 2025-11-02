pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "ishwaryamallesh/flaskproject"
    DOCKER_TAG = "${env.BUILD_NUMBER}"
    DOCKERHUB_CREDENTIALS = "dockerhub_creds"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install dependencies & Unit Tests') {
      steps {
        sh 'python3 -m venv .venv || true'
        sh '. .venv/bin/activate && pip install -r requirements.txt'
        sh '. .venv/bin/activate && pytest -q'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'sonar-scanner'
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          timeout(time: 2, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
              error "Pipeline failed due to quality gate: ${qg.status}"
            }
          }
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ."
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh "echo $PASS | docker login -u $USER --password-stdin"
          sh "docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
          sh "docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest"
          sh "docker push ${env.DOCKER_IMAGE}:latest"
          sh "docker logout"
        }
      }
    }

    stage('Deploy (local)') {
      steps {
        sh 'docker rm -f ci-cd-demo || true'
        sh "docker run -d --name ci-cd-demo -p 5000:5000 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
      }
    }
  }

  post {
    always {
      echo "Pipeline completed."
    }
  }
}
