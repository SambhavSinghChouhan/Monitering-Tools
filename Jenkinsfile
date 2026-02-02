pipeline {
    agent any

    environment {
        REPO_NAME = "Monitoring-Tools"
    }

    stages {

        stage('Checkout Repository') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Prepare System (Fix Jenkins Repo)') {
            steps {
                sh '''
                echo "Removing broken Jenkins APT repo if present..."
                sudo rm -f /etc/apt/sources.list.d/jenkins.list
                sudo rm -f /etc/apt/trusted.gpg.d/jenkins.gpg
                '''
            }
        }

        stage('Install Docker') {
            steps {
                sh '''
                if ! command -v docker >/dev/null 2>&1; then
                    echo "Docker not found. Installing Docker..."

                    sudo apt-get update -y
                    sudo apt-get install -y \
                        ca-certificates \
                        curl \
                        gnupg \
                        lsb-release

                    curl -fsSL https://get.docker.com | sudo sh
                else
                    echo "Docker already installed"
                fi
                '''
            }
        }

        stage('Configure Docker Permissions') {
            steps {
                sh '''
                echo "Configuring Docker permissions..."

                sudo usermod -aG docker jenkins || true
                sudo usermod -aG docker $USER || true

                sudo systemctl restart docker

                # CI safety net
                sudo chmod 666 /var/run/docker.sock
                '''
            }
        }

        stage('Install Docker Compose') {
            steps {
                sh '''
                if ! command -v docker-compose >/dev/null 2>&1; then
                    echo "Installing Docker Compose..."
                    sudo curl -L \
                      "https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-$(uname -s)-$(uname -m)" \
                      -o /usr/local/bin/docker-compose
                    sudo chmod +x /usr/local/bin/docker-compose
                else
                    echo "Docker Compose already installed"
                fi
                '''
            }
        }

        stage('Verify Docker Setup') {
            steps {
                sh '''
                docker --version
                docker compose version || docker-compose --version
                docker info
                '''
            }
        }

        stage('Start Monitoring Stack') {
            steps {
                sh '''
                echo "Starting monitoring stack..."
                docker compose up -d
                docker ps
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Monitoring stack is up and running!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}

