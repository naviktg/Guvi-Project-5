pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://docker:2376'
        DOCKER_TLS_VERIFY = '1'
        DOCKER_CERT_PATH = '/var/jenkins_home/.docker-certs'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/naviktg/Guvi-Project-5.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Unit Test') {
            steps {
                sh './mvnw test'
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true,
                  testResults: 'target/surefire-reports/*.xml'
        }
        success {
            echo 'Project 5 CI pipeline completed successfully!'
        }
        failure {
            echo 'Project 5 CI pipeline failed. Check console output.'
        }
    }
}
