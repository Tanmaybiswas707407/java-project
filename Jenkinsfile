
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        GIT_URL = 'https://github.com/Tanmaybiswas707407/java-project.git'
        DOCKER_IMAGE = 'tanmay477/myjava-image'
        DOCKER_CREDENTIALS = '82377f3d-eaad-4d26-8e68-bdb06c13e4d8'
    }

    stages {

        stage('Pull Source Code') {
            steps {
                git branch: 'main', url: "${GIT_URL}"
            }
        }

        stage('Maven Build') {
            steps {
                sh '''
                    set -e
                    mvn clean package
                    cp target/*.jar .
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Docker Hub Login and Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    set -e

                    docker rm -f myjava-app || true

                    docker run -d \
                        --restart unless-stopped \
                        --name myjava-app \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }
    }
}
