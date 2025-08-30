pipeline {
    agent any

    environment {
        // Using your personalized Docker Hub repository
        DOCKER_IMAGE = "syaddevops/devopsexamapp:latest" 
        // Correctly referencing the SonarQube scanner tool
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                // Using your forked GitHub repository
                git url: 'https://github.com/syaddays/devops-exam-app.git', 
                    branch: 'master'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // This stage was missing from some versions. It's included here.
                // Assumes SonarQube is configured in Jenkins with the name 'sonar'
                withSonarQubeEnv('sonar') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }

        stage('Verify Docker Compose') {
            steps {
                sh '''
                docker compose version || { echo "Docker Compose not available"; exit 1; }
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('backend') {
                    script {
                        // CORRECTED: Using credentialsId 'docker' as per the tutorial
                        withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                            sh "docker build -t ${DOCKER_IMAGE} ."
                        }
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // CORRECTED: Using credentialsId 'docker' as per the tutorial
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                # Clean up any existing containers
                docker compose down --remove-orphans || true
                
                # Start services using the image pushed to Docker Hub
                # NOTE: The --build flag is removed. docker-compose.yml should now 
                # reference the image: syaddevops/devopsexamapp:latest
                docker compose up -d
                
                # Wait for MySQL to be ready
                echo "Waiting for MySQL to be ready..."
                timeout 120s bash -c '
                while ! docker compose exec -T mysql mysqladmin ping -uroot -prootpass --silent;
                do 
                    sleep 5;
                done'
                
                # Additional wait for full initialization
                sleep 10
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "=== Container Status ==="
                docker compose ps -a
                echo "=== Testing Flask Endpoint ==="
                curl -I http://localhost:5000 || true
                '''
            }
        }
    }

   post {
    success {
        // This block now only prints a success message.
        echo '🚀 Pipeline finished successfully!'
    }
    failure {
        // This block runs if any main stage fails.
        echo '❗ Pipeline failed. Check logs for errors.'
        // This command helps debug failures by showing container logs.
        sh 'docker compose logs --tail=50 || true'
    }
    always {
        // This block runs every time, after success or failure.
        // It's the perfect place for cleanup.
        echo 'Cleaning up workspace.'
        cleanWs()
    }
}
}
