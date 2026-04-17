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
                echo 'Génération des métadonnées pour Snyk...'
                /* --package-lock-only : Ne télécharge pas les fichiers, crée juste le plan.
                   --ignore-scripts    : Empêche de lancer "ng build" qui fait planter le stage.
                */
                sh 'docker run --rm -v ${WORKSPACE}:/app -w /app node:22-bookworm npm install --package-lock-only --ignore-scripts'
            }
        }

        stage('SAST Scan (Snyk)') {
            environment {
                SNYK_TOKEN = credentials('snyk-token')
            }
            steps {
                echo 'Analyse statique avec Snyk...'
                // On utilise les guillemets simples ' ' pour masquer le secret dans les logs
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
                echo 'Analyse Trivy...'
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image juice-shop-local'
            }
        }
    }
}
