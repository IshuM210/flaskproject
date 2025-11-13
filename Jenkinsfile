pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/IshuM210/flaskproject.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONARQUBE = credentials('sonartoken')
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=flaskproject \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://172.22.147.80:9000 \
                    -Dsonar.login=$SONARQUBE
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
    }
}
