pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ishwaryamallesh/flaskproject"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = "dockerhub_creds" // Jenkins credentials ID for DockerHub
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies & Unit Tests') {
            steps {
                sh '''
                    python3 -m venv .venv
                    source .venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pytest -q --junitxml=report.xml
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarserver') { // Name of SonarQube server in Jenkins
                    sh '''
                        source .venv/bin/activate
                        sonar-scanner -Dproject.settings=sonar-project.properties
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
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
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
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
                sh """
                    docker rm -f ci-cd-demo || true
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
