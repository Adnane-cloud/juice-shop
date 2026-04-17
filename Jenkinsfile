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
                echo 'Installation des dépendances pour le scan...'
                // On utilise un conteneur Node pour faire le npm install sans l'installer sur AWS
                sh 'docker run --rm -v ${WORKSPACE}:/app -w /app node:20-bookworm npm install --package-lock-only'
            }
        }

        stage('SAST Scan (Snyk)') {
            environment {
                SNYK_TOKEN = credentials('snyk-token')
            }
            steps {
                echo 'Analyse statique avec Snyk...'
                /* CORRECTIONS :
                   1. Utilisation de guillemets simples ' ' pour la sécurité ($SNYK_TOKEN au lieu de ${SNYK_TOKEN})
                   2. Ajout de --package-lock-only pour que Snyk puisse analyser sans tout télécharger
                */
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
