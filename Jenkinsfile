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

        stage('K8s Security Scan (Trivy)') {
            steps {
                echo 'Analyse de la configuration Kubernetes...'
                // Trivy peut scanner les fichiers YAML pour trouver des erreurs de sécurité
                sh 'docker run --rm -v ${WORKSPACE}:/app -w /app aquasec/trivy config .'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Déploiement sur le cluster K3s...'
                // 1. On crée le pont automatiquement dans le pipeline
                sh 'sudo docker save juice-shop-local:latest | sudo k3s ctr images import -'
                
                // 2. On applique le déploiement
                sh 'kubectl apply -f k8s/juice-shop.yaml'
                
                // 3. On force le redémarrage pour être sûr qu'il prenne la nouvelle image
                sh 'kubectl rollout restart deployment juice-shop'
                
                echo 'Application disponible sur le port 30001'
            }
        }
    }
}
