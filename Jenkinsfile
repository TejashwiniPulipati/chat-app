pipeline {
    agent any

    environment {
        BACKEND_ENV_FILE = "backend/.env"
        FRONTEND_ENV_FILE = "frontend/.env"
        REGISTRY_CREDENTIALS = "dockerhub-credentials"
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
                    // credentialsId: 'dockerhub-credentials' refers to the ID of the stored credentials in Jenkins.
                    // usernameVariable: 'DOCKER_USER' stores the Docker Hub username in DOCKER_USER.
                    // passwordVariable: 'DOCKER_PASS' stores the password in DOCKER_PASS.
                    // sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    // echo $DOCKER_PASS prints the password (without displaying it in logs).
                    // docker login -u $DOCKER_USER --password-stdin securely logs into Docker Hub using --password-stdin (recommended by Docker).
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker build -t pulipatitejashwini/chatapp-be:${APP_VERSION} backend/
                    docker build -t pulipatitejashwini/chatapp-fe:${APP_VERSION} frontend/
                    docker push pulipatitejashwini/chatapp-be:${APP_VERSION}
                    docker push pulipatitejashwini/chatapp-fe:${APP_VERSION}
                    """
                    }
                }
            }
        }

        stage('Deploy on Docker Server') {
            steps {
                script {
                    sh """
                    docker ps --filter "network=chatapp-network" -q | xargs -r docker rm -f
                    docker network rm -f chatapp-network || true
                    docker network create chatapp-network || true

                    docker container rm -f chatapp-db || true
                    docker run -dt --name chatapp-db -p 5432:5432 \
                        -e POSTGRES_USER=postgres \
                        -e POSTGRES_PASSWORD=app12345 \
                        -e POSTGRES_DB=chatappdb \
                        --network chatapp-network postgres

                    docker pull pulipatitejashwini/chatapp-be:${APP_VERSION}
                    docker container rm -f chatapp-be || true
                    docker run -dt --name chatapp-be -p 8081:8080 --env-file=${BACKEND_ENV_FILE} \
                        --network chatapp-network pulipatitejashwini/chatapp-be:${APP_VERSION}

                    docker pull pulipatitejashwini/chatapp-fe:${APP_VERSION}
                    docker container rm -f chatapp-fe || true
                    docker run -dt --name chatapp-fe -p 80:80 --env-file=${FRONTEND_ENV_FILE} \
                        --network chatapp-network pulipatitejashwini/chatapp-fe:${APP_VERSION}
                    """
                }
            }
        }
    }
}