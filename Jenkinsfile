pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rajath06/star-agile-health-care"
        DOCKER_TAG = "latest"
        K8S_NAMESPACE = "star-agile"
        GIT_REPO = "https://github.com/rajath06/star-agile-health-care.git"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }

        stage('Provision Test Cluster') {
            steps {
                sh 'terraform init && terraform apply -auto-approve'
            }
        }

        stage('Configure Test Cluster') {
            steps {
                sh 'ansible-playbook -i inventory setup.yml'
            }
        }

        stage('Deploy to Test Environment') {
            steps {
                sh "kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}"
            }
        }

        stage('Run Selenium Tests') {
            steps {
                sh 'mvn verify -Pselenium'
            }
        }

        stage('Approval for Production Deployment') {
            steps {
                script {
                    input message: 'Proceed to Production?', ok: 'Deploy'
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                sh "kubectl apply -f k8s/prod-deployment.yaml -n ${K8S_NAMESPACE}"
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Build Failed!'
        }
    }
}
