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
                // Use bash shell explicitly
                    sh '''#!/bin/bash
                           set -e
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
                withSonarQubeEnv('sonarserver') { // Name of your SonarQube server in Jenkins
                    sh '''#!/bin/bash
                        set -e
                        source .venv/bin/activate
                        /opt/sonar-scanner/bin/sonar-scanner -Dproject.settings=sonar-project.properties
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
                sh '''#!/bin/bash
                    set -e
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''#!/bin/bash
                        set -e
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy (local)') {
            steps {
                sh '''#!/bin/bash
                    set -e
                    docker rm -f ci-cd-demo || true
                    docker run -d --name ci-cd-demo -p 5000:5000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
    }
}
