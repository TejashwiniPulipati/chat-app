pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pulipatitejashwini/chat-fe"
        REGISTRY_CREDENTIALS = "dockerhub-credentials"
        DOCKER_SERVER = "18.134.226.211"
    }

    stages {
        stage('code-analysis') {
            steps {
                echo 'Sonar Analysis Started'
                sh 'cd frontend && docker run --rm -e SONAR_HOST_URL="http://3.9.144.17:9000" -v ".:/usr/src" -e SONAR_TOKEN="sqp_6995bce8d9daab4353dfb2944c4f0ea793d78f35" sonarsource/sonar-scanner-cli -Dsonar.projectKey=chat'
                echo 'Sonar Analysis Completed'
            }
        }

        stage('Extract Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    env.APP_VERSION = packageJson.version
                    echo "${APP_VERSION}"
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE}:${APP_VERSION} .
                    docker login -u pulipatitejashwini -p Npnt@2412
                    docker push ${DOCKER_IMAGE}:${APP_VERSION}
                    """
                }
            }
        }

        stage('Deploy on Docker Server') {
            steps {
                script {
                    sh """
                    ssh ${DOCKER_SERVER} "
                    docker pull ${DOCKER_IMAGE}:${APP_VERSION} &&
                    docker stop chat-fe || true &&
                    docker rm chat-fe || true &&
                    docker run -d --name chatapp-fe -p 80:80 ${DOCKER_IMAGE}:${APP_VERSION}
                    "
                    """
                }
            }
        }
    }
}