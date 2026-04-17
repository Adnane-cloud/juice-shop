pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code depuis GitHub...'
                checkout scm
            }
        }
        
        stage('SAST Scan (Snyk)') {
            environment {
                // Jenkins récupère automatiquement ton secret ici
                SNYK_TOKEN = credentials('snyk-token')
            }
            steps {
                echo 'Analyse du code source avec Snyk...'
                // On utilise un conteneur Snyk pour ne rien avoir à installer sur ton serveur
                sh "docker run --rm -e SNYK_TOKEN=${SNYK_TOKEN} -v ${WORKSPACE}:/app snyk/snyk:node /app snyk test || true"
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
                echo 'Analyse des vulnérabilités de l image avec Trivy...'
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image juice-shop-local'
            }
        }
    }
}
