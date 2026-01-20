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
                    docker rm -f test-container || true

                    docker run -d --name test-container \
                      $DOCKER_USER/$IMAGE_NAME:${BUILD_NUMBER}

                    echo "Czekam aż aplikacja wstanie..."

                    for i in $(seq 1 10); do
                      if docker exec test-container curl -f http://localhost:5001; then
                        echo "Aplikacja działa"
                        break
                      fi
                      sleep 2
                    done

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

        stage('Update Helm Chart') {
            steps {
                sh '''
                    git clone https://github.com/kamil38490/aplikacja1-helm.git helm-temp
                    cd helm-temp/aplikacja1

                    sed -i "s/tag:.*/tag: ${BUILD_NUMBER}/" values.yaml

                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins CI"
                    git add values.yaml
                    git commit -m "Update image tag to ${BUILD_NUMBER}"
                    git push origin main
                '''
            }
        }
    } // koniec stages

    post {
        always {
            sh 'docker logout'
        }
    }
} // koniec pipeline

