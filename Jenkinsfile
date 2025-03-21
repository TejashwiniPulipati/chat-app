 pipeline {
    agent any

    environment {
        REGISTRY_CREDENTIALS = "dockerhub-credentials"
        NETWORK_NAME = "myapp-network"
    }

    stages {
        stage('Extract Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'frontend/package.json'
                    env.APP_VERSION = packageJson.version
                    echo "App Version: ${APP_VERSION}"
                }
            }
        }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: REGISTRY_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        
                        docker build -t pulipatitejashwini/myapp-fe:${APP_VERSION} frontend/
                        docker build -t pulipatitejashwini/myapp-be:${APP_VERSION} backend/

                        docker push pulipatitejashwini/myapp-fe:${APP_VERSION}
                        docker push pulipatitejashwini/myapp-be:${APP_VERSION}
                        """
                    }
                }
            }
        }

        stage('Deploy on Docker Server') {
            steps {
                script {
                    sh """
                    # Stop and remove old containers in the network
                    docker ps --filter "network=${NETWORK_NAME}" -q | xargs -r docker rm -f
                    docker network rm ${NETWORK_NAME} || true
                    docker network create ${NETWORK_NAME} || true

                    # Start Database Container
                    docker container rm -f myapp-db || true
                    docker run -dt --name myapp-db -p 5432:5432 \
                        -e POSTGRES_USER=postgres \
                        -e POSTGRES_PASSWORD=app12345 \
                        -e POSTGRES_DB=myappdb \
                        --network ${NETWORK_NAME} postgres

                    # Start Backend Container
                    docker pull pulipatitejashwini/myapp-be:${APP_VERSION}
                    docker container rm -f myapp-be || true
                    docker run -dt --name myapp-be -p 8081:8080 \
                        -e DATABASE_URL="postgresql://postgres:app12345@myapp-db:5432/myappdb" \
                        --network ${NETWORK_NAME} pulipatitejashwini/myapp-be:${APP_VERSION}

                    # Start Frontend Container
                    docker pull pulipatitejashwini/myapp-fe:${APP_VERSION}
                    docker container rm -f myapp-fe || true
                    docker run -dt --name myapp-fe -p 80:80 \
                        --network ${NETWORK_NAME} pulipatitejashwini/myapp-fe:${APP_VERSION}
                    """
                }
            }
        }
    }
}