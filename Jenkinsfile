pipeline {
    agent any

    environment {
        IMAGE_NAME = "test"
    }

    stages {

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker_hub', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build \
                      -f docker/Dockerfile \
                      -t $DOCKER_USER/$IMAGE_NAME:${BUILD_NUMBER} \
                      -t $DOCKER_USER/$IMAGE_NAME:latest \
                      docker
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker push $DOCKER_USER/$IMAGE_NAME:${BUILD_NUMBER}
                    docker push $DOCKER_USER/$IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
