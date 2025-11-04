pipeline {
  agent any
  environment {
    DOCKERHUB_CRED = credentials('dockerhub')
    IMAGE = "docker.io/jhimanshu50/bankpro-core:latest"
  }
  stages {
    stage('Checkout') {
      steps { git branch: 'main', url: 'https://github.com/himanshujoshi50/bankpro-core.git' }
    }
  stage('Build & Test') {
  steps {
    sh 'mvn -B -DskipTests=false clean package'
  }
}

    stage('Docker Build') {
      steps { sh 'docker build -t $IMAGE .' }
    }
    stage('Docker Login & Push') {
      steps {
        sh 'echo "$DOCKERHUB_CRED_PSW" | docker login -u "$DOCKERHUB_CRED_USR" --password-stdin'
        sh 'docker push $IMAGE'
      }
    }
    stage('Deploy AWS') {
      steps {
        sshagent(credentials: ['ssh-aws-ubuntu']) {
          sh '''
            ansible-playbook -i ansible/inventory.ini ansible/deploy.yml             --limit aws -e app_image=$IMAGE
          '''
        }
      }
    }
    stage('Deploy Azure') {
      steps {
        sshagent(credentials: ['ssh-azure-azureuser']) {
          sh '''
            ansible-playbook -i ansible/inventory.ini ansible/deploy.yml             --limit azure -e app_image=$IMAGE
          '''
        }
      }
    }
  }
  post {
    always {
      sh 'docker logout || true'
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
  }
}
