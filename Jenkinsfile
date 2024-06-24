pipeline {
  agent any
  environment {
        KUBECONFIG = credentials('kubeconfig')
    }
  stages {
    stage('Cleaning Workspace') {
      steps {
        cleanWs()
      }
    }
    stage('Clone Repository') {
      steps {
        git(url: 'https://github.com/muhd-zunurain/microservice.git', branch: 'main')
      }
    }
    stage('Filesystem Check') {
      steps {
        sh 'trivy fs --skip-db-update . > /var/lib/jenkins/security_report/filesystem_report.txt'
        sh 'cat /var/lib/jenkins/security_report/filesystem_report.txt | grep Total'
      }
    }

    stage('Replace Placholder') {
      steps {
        sh "sed -i 's/@BUILD_NUMBER/$BUILD_NUMBER/g' docker-compose.yml"
      }
    }
    
    stage('Building Images and Starting Containers') {
      steps {
        sh 'docker compose up -d'
      }
    }
        stage('Image Vulnerabilities Check') {
      steps {
        sh 'trivy image --skip-db-update shahrilx/frontend > /var/lib/jenkins/security_report/frontend-vul.txt'
        sh 'cat /var/lib/jenkins/security_report/frontend-vul.txt | grep Total'
        sh 'trivy image --skip-db-update shahrilx/api > /var/lib/jenkins/security_report/api-vul.txt'
        sh 'cat /var/lib/jenkins/security_report/api-vul.txt | grep Total'
        sh 'trivy image --skip-db-update shahrilx/quotes > /var/lib/jenkins/security_report/quotes-vul.txt'
        sh 'cat /var/lib/jenkins/security_report/quotes-vul.txt | grep Total'
      }
    }
    stage('Testing The Apps') {
      steps {
        sleep(5)
        sh 'chmod +x testing_phase.sh'
        sh './testing_phase.sh'
      }
    }

    stage('Push Image') {
      steps {
        sh "docker push 128.199.97.48:31735/frontend:$BUILD_NUMBER && docker push 128.199.97.48:31735/api:$BUILD_NUMBER && docker push 128.199.97.48:31735/quotes:$BUILD_NUMBER"
    }
    }
    stage('Cleaning Test Environment') {
      steps {
        sh 'docker compose down'
        sh "docker rmi 128.199.97.48:31735/frontend:$BUILD_NUMBER"
        sh "docker rmi 128.199.97.48:31735/api:$BUILD_NUMBER"
        sh "docker rmi 128.199.97.48:31735/quotes:$BUILD_NUMBER"
      }
    }
    stage('Deploy To Production') {
      steps {
        sh 'kubectl apply -f kubernetes/.'
      }
    }

  }
}
