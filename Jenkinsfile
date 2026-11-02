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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Project5') {
                    sh '''
                        ./mvnw sonar:sonar \
                            -Dsonar.projectKey=habit-tracker \
                            -Dsonar.projectName="Habit Tracker"
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t navikaran/habit-tracker:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-project5',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push navikaran/habit-tracker:${BUILD_NUMBER}
                        docker logout
                    '''
                }
            }
        }

        stage('Update Helm Configuration') {
            steps {
                sh '''
                    sed -i "s/^  tag:.*/  tag: \\"${BUILD_NUMBER}\\"/" helm/habit-tracker/values.yaml
                    grep "tag:" helm/habit-tracker/values.yaml
                '''
            }
        }

        stage('Push Deployment Config to GitHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-project5',
                    usernameVariable: 'GITHUB_USER',
                    passwordVariable: 'GITHUB_TOKEN'
                )]) {
                    sh '''
                        set +x

                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@users.noreply.github.com"

                        git add helm/habit-tracker/values.yaml

                        if ! git diff --cached --quiet; then
                            git commit -m "Update Habit Tracker image to build ${BUILD_NUMBER}"

                            AUTH=$(printf "%s:%s" "$GITHUB_USER" "$GITHUB_TOKEN" | base64 -w0)

                            git -c http.extraheader="AUTHORIZATION: basic $AUTH" push origin HEAD:main
                        else
                            echo "No Helm configuration changes to push."
                        fi
                    '''
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true,
                  testResults: 'target/surefire-reports/*.xml'
        }

        success {
            echo 'Project 5 CI and GitOps configuration update completed successfully!'
        }

        failure {
            echo 'Project 5 CI pipeline failed. Check console output.'
        }
    }
}
