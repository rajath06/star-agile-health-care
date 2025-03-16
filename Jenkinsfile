pipeline {
  agent any

  environment {
    M2_HOME = '/usr/share/maven'
    PATH = "$M2_HOME/bin:$PATH"
  }

  stages {  
    stage('Install Maven') {
      steps {
        sh 'sudo apt update && sudo apt install -y maven'
        sh 'mvn -version'
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
        sh 'docker build -t rajath06/health-me-app:4.0 .'
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
        sh 'docker push rajath06/health-me-app:4.0'
      }
    }

    stage('Ansible Deployment') {
      steps {
        ansiblePlaybook credentialsId: 'ansible-ssh', disableHostKeyChecking: true, 
        installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible.yml', vaultTmpPath: ''
      }
    }
  }
}









