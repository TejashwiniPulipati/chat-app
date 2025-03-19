pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pulipatitejashwini/chat-fe"
        REGISTRY_CREDENTIALS = "dockerhub-credentials"
        DOCKER_SERVER = "18.134.226.211"
    }

    stages {
        stage('Extract Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'frontend/package.json'
                    env.APP_VERSION = packageJson.version
                    echo "${APP_VERSION}"
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${DOCKER_IMAGE}:${APP_VERSION} frontend/
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