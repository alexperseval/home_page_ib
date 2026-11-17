
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'ibcgos-frontend-alexandre'
        IMAGE_TAG = '1.0'

        // Remplacer par ton username Docker Hub
        DOCKERHUB_USERNAME = 'alexperseval'

        CONTAINER_NAME = 'ibcgos-container-alexandre'
        HOST_PORT = '8061'
        CONTAINER_PORT = '80'

        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-alexandre'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Recuperation du code source...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Construction de l image Docker...'

                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('DockerHub Login') {
            steps {
                echo 'Authentification sur Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS}",
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        printf '%s' "$DH_TOKEN" | docker login \
                          -u "$DH_USER" \
                          --password-stdin
                    '''
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                echo 'Creation du tag Docker Hub...'

                sh '''
                    docker tag \
                      ${IMAGE_NAME}:${IMAGE_TAG} \
                      ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Publication de l image sur Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS}",
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        printf '%s' "$DH_TOKEN" | docker login \
                          -u "$DH_USER" \
                          --password-stdin

                        docker push \
                          ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Stop Old Container') {
            steps {
                echo 'Arret de l ancienne instance...'

                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo 'Demarrage du nouveau conteneur...'

                sh '''
                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p ${HOST_PORT}:${CONTAINER_PORT} \
                      ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verification du deploiement...'

                sh '''
                    sleep 5

                    docker ps --filter "name=${CONTAINER_NAME}"

                    curl -f http://localhost:${HOST_PORT}
                '''
            }
        }
    }

    post {
        success {
            echo 'Application IBCGOS deployee avec succes !'
            echo 'Image publiee sur Docker Hub.'
            echo 'URL : http://jenkins.sip-academy.com:8059'
        }

        failure {
            echo 'Erreur pendant la pipeline CI/CD.'
        }

        always {
            echo 'Fin de la pipeline Jenkins.'
        }
    }
}