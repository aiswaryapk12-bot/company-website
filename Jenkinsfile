pipeline {
    agent none

    environment {
        DOCKERHUB_USERNAME = 'aiswaryapk1234'
        IMAGE_NAME = 'company-website'
        CONTAINER_NAME = 'company-landing-page'
        PORT = '9090'
    }

    stages {

        // Stage 1: Checkout and build on build-server
        stage('Checkout and Build') {
            agent {
                label 'build-server'
            }

            steps {
                echo 'Checking out source code...'
                checkout scm

                echo 'Building Docker image...'

                sh '''
                    docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        // Stage 2: Push image to Docker Hub
        stage('Push to Docker Hub') {
            agent {
                label 'build-server'
            }

            steps {
                echo 'Logging in to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }

        // Stage 3: Deploy on deploy-server
        stage('Deploy Application') {
            agent {
                label 'deployment-server'
            }

            steps {
                echo 'Deploying application on deploy-server...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker pull ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}

                        docker rm -f ${CONTAINER_NAME} || true

                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p ${PORT}:80 \
                            --restart unless-stopped \
                            ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully!'
            echo "Docker image: ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}"
            echo "Website: http://<deploy-server-ip>:${PORT}"
        }

        failure {
            echo 'Pipeline failed. Please check Jenkins logs.'
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}
