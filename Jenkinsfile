pipeline {
    agent any

    environment {
        IMAGE_NAME = "test-node"
        DOCKER_USER = 'kamil38490'
        HELM_REPO = 'https://github.com/Kamil38490/argo_cd.git'
    }

    stages {

        // ===== Etap 1: Budowa Dockera tylko przy zmianach w kodzie aplikacji =====
        stage('Build Docker Image') {
            when {
                anyOf {
                    changeset "docker/**"
                    changeset "app/**"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }

                sh '''
                    docker build -f docker/Dockerfile \
                        -t $DOCKER_USER/$IMAGE_NAME:${BUILD_NUMBER} \
                        -t $DOCKER_USER/$IMAGE_NAME:latest \
                        docker
                '''

                sh '''
                    docker push $DOCKER_USER/$IMAGE_NAME:${BUILD_NUMBER}
                    docker push $DOCKER_USER/$IMAGE_NAME:latest
                '''
            }
        }

        // ===== Etap 2: Aktualizacja Helm chartu =====
        stage('Update Helm Chart') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github_token', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh '''
                    # Czyści stare repo Helm
                    rm -rf argo-temp

                    # Klonuje repo Helm z tokenem
                    git clone https://$GIT_USER:$GIT_TOKEN@github.com/Kamil38490/argo_cd.git argo-temp
                    cd argo-temp/aplikacja1

                    # Aktualizacja tagu tylko jeśli zmienił się obraz Dockera
                    sed -i "s/tag:.*/tag: ${BUILD_NUMBER}/" values.yaml

                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins CI"

                    # Commit z [ci skip] żeby nie uruchamiać pętli w Jenkins
                    git add values.yaml
                    git commit -m "Update image tag to ${BUILD_NUMBER} [ci skip]" || true
                    git push origin main
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}

