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
                // Utilisation du token stocké dans les credentials Jenkins
                SNYK_TOKEN = credentials('snyk-token')
            }
            steps {
                echo 'Analyse statique du code source avec Snyk...'
                /* CORRECTION : 
                   -w /app : Définit le dossier de travail pour que Snyk trouve le code.
                   || true : Empêche le build d'échouer à cause des nombreuses failles de Juice Shop.
                */
                sh "docker run --rm -e SNYK_TOKEN=${SNYK_TOKEN} -v ${WORKSPACE}:/app -w /app snyk/snyk:node snyk test || true"
            }
        }

        stage('Build Image') {
            steps {
                echo 'Construction de l image Docker Juice Shop...'
                // Construction de l'image locale
                sh 'docker build -t juice-shop-local .'
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                echo 'Analyse des vulnérabilités de l image finale avec Trivy...'
                /* Trivy scanne l'image Docker pour trouver des vulnérabilités (CVE) 
                   dans l'OS et les packages installés.
                */
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image juice-shop-local'
            }
        }
    }

    post {
        always {
            echo 'Nettoyage facultatif ou notifications ici.'
        }
    }
}
