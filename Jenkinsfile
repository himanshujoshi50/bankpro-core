pipeline {
    agent any

    environment {
        IMAGE = "jhimanshu50/bankpro-core:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $IMAGE ."
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $IMAGE
                    """
                }
            }
        }

        stage('Deploy AWS') {
            steps {
                sshagent(credentials: ['ssh-aws-ubuntu']) {
                    sh '''
                        ansible-playbook -i ansible/inventory.ini ansible/deploy.yml --limit aws -e app_image=$IMAGE
                        rc=$?
                        if [ $rc -eq 0 ] || [ $rc -eq 2 ]; then exit 0; else exit $rc; fi
                    '''
                }
            }
        }

        stage('Deploy Azure') {
            steps {
                sshagent(credentials: ['ssh-azure-azureuser']) {
                    sh '''
                        ansible-playbook -i ansible/inventory.ini ansible/deploy.yml --limit azure -e app_image=$IMAGE
                        rc=$?
                        if [ $rc -eq 0 ] || [ $rc -eq 2 ]; then exit 0; else exit $rc; fi
                    '''
                }
            }
        }
    }

    post {
        always {
            sh "docker logout"
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
    }
}
