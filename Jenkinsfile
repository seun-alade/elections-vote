pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "alade01"
        DOCKER_IMAGE = "vote"  // Change to your Docker Hub repo name
        DOCKER_TAG = "latest"
        DOCKER_HUB_CREDENTIALS = "docker-hub-credentials"  // Set this in Jenkins credentials
        EC2_INSTANCE = "ubuntu@15.156.204.136"  // Change to your EC2 public IP
        SSH_CREDENTIALS = "ec2-ssh-key"  // Set this in Jenkins credentials
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/seun-alade/elections-vote.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t \$DOCKERHUB_USERNAME/$DOCKER_IMAGE:\$DOCKER_TAG .
                """
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker push \$DOCKERHUB_USERNAME/$DOCKER_IMAGE:\$DOCKER_TAG
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent(['ec2-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no $EC2_INSTANCE << EOF
                            
                            echo "Pulling latest image..."
                            docker pull $DOCKERHUB_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG
                            
                            echo "Stopping old container if it exists..."
                            docker stop vote || true
                            docker rm vote || true
    
                            echo "Running new container..."
                            docker run -d --name vote --network front-tier --network back-tier -p 5000:80 $DOCKERHUB_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG
EOF
                        """
                    }
                }
            }
        }
    }
}