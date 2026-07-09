pipeline {
    agent any

    environment {
        PROJECT_ID  = 'project-6889e356-60dc-498f-b12'
        REGION      = 'northamerica-northeast2'
        REPO        = 'my-repo'
        IMAGE_NAME  = 'my-app'
        IMAGE_TAG   = "${env.BUILD_NUMBER}"
        IMAGE       = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
        CLUSTER     = 'my-gke-cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE} ."
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                sh '''
                    gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet
                    docker push ${IMAGE}
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                    gcloud container clusters get-credentials ${CLUSTER} --region ${REGION} --project ${PROJECT_ID}
                    sed "s|PLACEHOLDER|${IMAGE}|g" k8s/deployment.yaml > k8s/deployment-final.yaml
                    kubectl apply -f k8s/deployment-final.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/my-app-deployment
                '''
            }
        }
    }

    post {
        success {
            echo "Deployed ${IMAGE} successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
