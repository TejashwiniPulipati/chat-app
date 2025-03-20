pipeline {
    agent any

    environment {
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
                    docker container run -dt --name chatapp-db -p 5432:5432 postgres
                    docker pull pulipatitejashwini/chatapp-be:${APP_VERSION} 
                    docker container run -dt --name chatapp-be -p 8081:8080 pulipatitejashwini/chatapp-be:${APP_VERSION}
                    docker pull pulipatitejashwini/chatapp-fe:${APP_VERSION} 
                    docker container run -dt --name chatapp-fe -p 80:80 pulipatitejashwini/chatapp-fe:${APP_VERSION}
                    """
                }
            }
        }
    }
}