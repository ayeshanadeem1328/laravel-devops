pipeline {
    agent any
    environment {
        IMAGE = "laravel-app:${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build Docker Image') {
            steps { sh 'docker build -t $IMAGE .' }
        }
        stage('Load Image into Minikube') {
            steps { sh 'minikube image load $IMAGE' }
        }
        stage('Deploy Database') {
            steps {
                sh '''
                kubectl apply -f k8s/secret.yaml
                kubectl apply -f k8s/mysql.yaml
                kubectl rollout status deployment/mysql --timeout=240s
                '''
            }
        }
        stage('Deploy Laravel') {
            steps {
                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|$IMAGE|" k8s/laravel.yaml
                kubectl apply -f k8s/laravel.yaml
                kubectl rollout status deployment/laravel-app --timeout=240s
                '''
            }
        }
        stage('Run Migrations') {
            steps { sh 'kubectl exec deploy/laravel-app -- php artisan migrate --force' }
        }
        stage('Verify') {
            steps {
                sh '''
                kubectl get pods,svc
                curl -sI http://$(minikube ip):30080 | head -1
                '''
            }
        }
    }
}
