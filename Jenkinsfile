pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Préparation des métadonnées...'
                sh 'docker run --rm -v ${WORKSPACE}:/app -w /app node:22-bookworm npm install --package-lock-only --ignore-scripts'
            }
        }

        stage('SAST Scan (Snyk)') {
            environment {
                SNYK_TOKEN = credentials('snyk-token')
            }
            steps {
                echo 'Audit du code source (Snyk)...'
                sh 'docker run --rm -e SNYK_TOKEN=$SNYK_TOKEN -v ${WORKSPACE}:/app -w /app snyk/snyk:node snyk test --package-lock-only || true'
            }
        }

        stage('Build Image') {
            steps {
                echo 'Construction de l image Docker...'
                sh 'docker build -t juice-shop-local .'
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                echo 'Audit de l image (Trivy)...'
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image juice-shop-local || true'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Déploiement en cours sur le port 8001...'
                /* 1. On arrête et supprime l'ancien conteneur s'il existe pour éviter les conflits */
                sh 'docker stop juice-shop-app || true'
                sh 'docker rm juice-shop-app || true'
                
                /* 2. On lance le nouveau conteneur en arrière-plan (-d)
                   Le port interne de Juice Shop est 3000, on l'expose sur le 8001 */
                sh 'docker run -d --name juice-shop-app -p 8001:3000 juice-shop-local'
                
                echo 'Déploiement réussi ! L application est en ligne.'
            }
        }
    }
}
