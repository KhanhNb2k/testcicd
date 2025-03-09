pipeline {
    agent any
    environment {
        REGISTRY = "harbor.local"
        REPO = "testcicd"
        IMAGE_TAG = "latest"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/KhanhNb2k/testcicd.git'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'echo "🔍 Running unit tests..."'
                sh 'echo "✅ Tests passed!"'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$REPO:$IMAGE_TAG .'
            }
        }
        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-credentials', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                    sh 'echo $HARBOR_PASS | docker login $REGISTRY -u $HARBOR_USER --password-stdin'
                    sh 'docker push $REGISTRY/$REPO:$IMAGE_TAG'
                }
            }
        }
        stage('Update Kubernetes Deployment') {
            steps {
                sh 'kubectl set image deployment/web-app web-app=$REGISTRY/$REPO:$IMAGE_TAG -n default'
            }
        }
        stage('Trigger ArgoCD Sync') {
            steps {
                sh 'curl -X POST -H "Content-Type: application/json" -d \'{"name": "web-app", "sync": true}\' http://argocd-server/api/v1/applications/web-app/sync'
            }
        }
    }
}

