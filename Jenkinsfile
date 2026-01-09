pipeline {
    agent any

    environment {
        IMAGE_NAME = "test-node"
        DOCKER_USER = 'kamil38490'
    }

    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
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

        stage('Test Image') {
            steps {
                sh '''
                    # sprzątanie po poprzednich buildach
                    docker rm -f test-container || true

                    # start kontenera
                    docker run -d --name test-container -p 5001:5001 \
                      $DOCKER_USER/$IMAGE_NAME:${BUILD_NUMBER}

                    # czekamy aż aplikacja wstanie
                    for i in {1..10}; do
                      curl -f http://localhost:5001 && break
                      sleep 2
                    done

                    # zatrzymanie kontenera
                    docker stop test-container
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

