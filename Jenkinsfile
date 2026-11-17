
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'ibcgos-frontend-alexandre'
        IMAGE_TAG = '1.0'
        CONTAINER_NAME = 'ibcgos-container-alexandre'
        HOST_PORT = '8061'
        CONTAINER_PORT = '80'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Récupération du code source...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Construction de l image Docker...'
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Stop Old Container') {
            steps {
                echo 'Arrêt de l ancienne instance...'
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo 'Démarrage du nouveau conteneur...'
                sh '''
                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p ${HOST_PORT}:${CONTAINER_PORT} \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Vérification du déploiement...'
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
            echo 'Application IBCGOS déployée avec succès !'
            echo 'URL : https://jenkins.sip-academy.com:8059'
        }

        failure {
            echo 'Erreur pendant le déploiement de IBCGOS.'
        }

        always {
            echo 'Fin de la pipeline Jenkins.'
        }
    }
}
