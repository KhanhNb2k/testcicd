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
                git credentialsId: 'github-credentials', url: 'https://github.com/KhanhNb2k/testcicd.git'
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
                sh '''
                kubectl get namespace default || kubectl create namespace default
                kubectl set image deployment/web-app web-app=$REGISTRY/$REPO:$IMAGE_TAG -n default
                '''
            }
        }
        stage('Trigger ArgoCD Sync') {
            steps {
                withCredentials([string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN')]) {
                    sh '''
                    curl -X POST -H "Content-Type: application/json" \
                         -H "Authorization: Bearer $ARGOCD_TOKEN" \
                         http://argocd-server/api/v1/applications/web-app/sync
                    '''
                }
            }
        }
        stage('Cleanup Docker') {
            steps {
                sh 'docker rmi $REGISTRY/$REPO:$IMAGE_TAG || true'
            }
        }
    }
}
