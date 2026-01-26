pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "nareshbabu1991/movie-app"
        IMAGE_TAG  = "build-${BUILD_NUMBER}"

        GIT_USER  = "Velocity9919"
        GIT_EMAIL = "ynareshbabu1992@gmail.com"
        GIT_REPO  = "github.com/Velocity9919/Production-Grade-App.git"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'main'
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set -e
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            when {
                branch 'main'
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    sh '''
                        set -e

                        git config user.name "${GIT_USER}"
                        git config user.email "${GIT_EMAIL}"

                        git checkout main
                        git pull origin main

                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                        git add k8s/deployment.yml
                        git commit -m "Update image to ${IMAGE_TAG}" || echo "No changes to commit"

                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@${GIT_REPO} main
                    '''
                }
            }
        }
    }

    post {
        always {
            docker logout || true
        }
    }
}
