pipeline {
  agent any

  environment {
    M2_HOME = '/usr/share/maven'
    PATH = "$M2_HOME/bin:$PATH"
    DOCKER_IMAGE = "rajath06/healthcare-app:1.0"
    KUBE_CONFIG = "/home/ubuntu/.kube/config"
  }

  stages {  
    stage('Install Dependencies') {
      steps {
        sh 'sudo apt update && sudo apt install -y maven docker.io'
        sh 'mvn -version'
        sh 'docker --version'
      }
    }

    stage('Create a Package') {
      steps {
         echo 'This will create a package using Maven'
         sh 'mvn package'
      }
    }

    stage('Publish the HTML Reports') {
      steps {
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, 
          reportDir: '/var/lib/jenkins/workspace/Star-Agile-Health-Care/target/surefire-reports', 
          reportFiles: 'index.html', reportName: 'HTML Report', 
          reportTitles: '', useWrapperFileDirectly: true])
      }
    }

    stage('Create a Docker Image') {
      steps {
        sh 'docker build -t rajath06/healthcare-app:1.0 .'
      }
    }

    stage('Login to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerlogin', passwordVariable: 'dockerpass', usernameVariable: 'dockeruser')]) {
          sh 'docker login -u ${dockeruser} -p ${dockerpass}'
        }
      }
    }

    stage('Push the Docker Image') {
      steps {
        sh 'docker push rajath06/healthcare-app:1.0'
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        echo 'Deploying to Kubernetes Cluster...'
        withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://your-k8s-master-ip:6443']) {
          sh 'kubectl apply -f k8s/deployment.yaml'
          sh 'kubectl apply -f k8s/service.yaml'
        }
      }
    }

    stage('Ansible Deployment') {
      steps {
        ansiblePlaybook credentialsId: 'ansible-ssh', disableHostKeyChecking: true, 
        installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible.yml', vaultTmpPath: ''
      }
    }

    stage('Verify Deployment') {
      steps {
        echo 'Checking Kubernetes deployment status...'
        sh 'kubectl get pods'
        sh 'kubectl get services'
      }
    }
  }
}
