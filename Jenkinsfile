pipeline {
    agent any

    environment {
        DOCKER_IMAGE1 = "node"
        DOCKER_IMAGE2 = "nginx"
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
                    docker build -t ${DOCKER_IMAGE1}:${APP_VERSION} backend/
                    docker build -t ${DOCKER_IMAGE2}:${APP_VERSION} frontend/
                    docker login -u pulipatitejashwini -p Npnt@2412
                    docker push ${DOCKER_IMAGE1}:${APP_VERSION}
                    docker push ${DOCKER_IMAGE2}:${APP_VERSION}
                    """
                }
            }
        }

        stage('Deploy on Docker Server') {
            steps {
                script {
                    sh """
                    ssh ${DOCKER_SERVER} "
                    docker pull ${DOCKER_IMAGE1}:${APP_VERSION} &&
                    docker run -d --name chatapp-fe -p 8081:8080 ${DOCKER_IMAGE1}:${APP_VERSION}
                    docker pull ${DOCKER_IMAGE2}:${APP_VERSION} &&
                    docker run -d --name chatapp-fe -p 80:80 ${DOCKER_IMAGE2}:${APP_VERSION}
                    "
                    """
                }
            }
        }
    }
}