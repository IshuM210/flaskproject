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
        bat '''
          "C:\\Users\\ishum\\AppData\\Local\\Programs\\Python\\Python313\\python.exe" -m venv .venv
          call .venv\\Scripts\\activate
          pip install -r requirements.txt
          pytest -q
        '''
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonarserver') {
          bat '''
            call .venv\\Scripts\\activate
            sonar-scanner -Dproject.settings=sonar-project.properties
          '''
        }
      }
    }

    stage('Quality Gate') {
      steps {
        script {
          timeout(time: 5, unit: 'MINUTES') {
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
        bat "docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ."
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          bat """
            echo %PASS% | docker login -u %USER% --password-stdin
            docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
            docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest
            docker push ${env.DOCKER_IMAGE}:latest
            docker logout
          """
        }
      }
    }

    stage('Deploy (local)') {
      steps {
        bat """
          docker rm -f ci-cd-demo || exit 0
          docker run -d --name ci-cd-demo -p 5000:5000 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
        """
      }
    }
  }

  post {
    always {
      echo "Pipeline completed."
    }
  }
}
