pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code depuis GitHub...'
                checkout scm
            }
        }
        stage('Build Image') {
            steps {
                echo 'Construction de l image Docker Juice Shop...'
                // On utilise --no-cache pour être sûr de repartir de zéro
                sh 'docker build -t juice-shop-local .'
            }
        }
        stage('Security Preview') {
            steps {
                echo 'Ici, nous ajouterons nos outils de scan (Trivy, Snyk...)'
                sh 'docker images | grep juice-shop-local'
            }
        }
    }
}
