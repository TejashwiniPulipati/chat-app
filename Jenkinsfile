pipeline {
    agent {
        docker { image 'nginx' }  
    }

    stages {
        stage('code-analysis') {
            steps {
                echo 'Sonar Analysis Started'
                sh 'cd frontend && sudo docker run --rm -e SONAR_HOST_URL="http://http://13.40.165.118:9000" -v ".:/usr/src" -e SONAR_TOKEN="sqp_cdb02aa58a3991153ea552ba063588526579f0f1" sonarsource/sonar-scanner-cli -Dsonar.projectKey=chat-app'
                echo 'Sonar Analysis Completed'
            }
        }
        stage('build containers') {
            steps {
                script {
                    def packageJson = readJSON file: 'chat-app/package.json'
                    def packageJSONVersion = packageJson.version
                    echo "${packageJSONVersion}"
                    sh 'cd chat-app/frontend && docker build -t pulipatitejashwini/chatapp1-be/${packageJSONVersion} .'
                    sh 'docker container run -dt --name chatapp-backend -p 80:80 pulipatitejashwini/chatapp1-be/${packageJSONVersion}'
                }
            }
        }
        stage('docker login') {
            steps {
                script {
                    echo 'logging into docker and pushing code to docker hub'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'password', usernameVariable: 'user')]) {
                    sh 'sudo docker login -u ${user} -p ${password}'

                    sh 'sudo docker push pulipatitejashwini/chatapp1-fe/${packageJSONVersion}'
                    }
                }
            }
        }
    }
}
