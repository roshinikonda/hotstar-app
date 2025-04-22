pipeline {
    agent any

    environment {
        IMAGE_NAME = 'yourdockerhubusername/hotstar-app'     // ✅ Change this
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'            // ✅ Jenkins credentials ID for DockerHub
        REMOTE_USER = 'ubuntu'                               // ✅ SSH username of your remote server
        REMOTE_HOST = 'your.remote.server.ip'                // ✅ Remote server's public IP or DNS
        REMOTE_SSH_KEY = 'remote-ssh-key-id'                 // ✅ Jenkins credentials ID for SSH private key
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Jenkins will automatically checkout the repo if Jenkinsfile is inside the repo
                echo "Code checked out from SCM."
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "Pushing Docker image to DockerHub..."
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS_ID,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy on Remote Server') {
            steps {
                echo "Deploying on remote server..."
                sshagent (credentials: [REMOTE_SSH_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST \\
                        'docker pull $IMAGE_NAME:$IMAGE_TAG && \\
                         docker stop hotstar || true && docker rm hotstar || true && \\
                         docker run -d --name hotstar -p 80:80 $IMAGE_NAME:$IMAGE_TAG'
                    """
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
