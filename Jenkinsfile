pipeline {
    agent { label "dev" }

    stages {
        stage("Code Clone") {
            steps {
                git url: "https://github.com/gopeshthakare13/two-tier-flask-app.git"
            }
        }

        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs . -o results.json"
            }
        }

        stage("Install Docker") {
            steps {
                sh '''
                  sudo apt update
                  sudo apt install -y docker.io docker-compose-v2
                  sudo usermod -aG docker ubuntu
                '''
            }
        }

        stage("Build Docker Image") {
            steps {
                sh "docker build -t two-tier-flask-app ."
            }
        }

        stage("Run Test Cases") {
            steps {
                echo "Developer writes test cases here..."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHubCreds", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_PASS")]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag two-tier-flask-app $DOCKER_USER/two-tier-flask-app:latest
                        docker push $DOCKER_USER/two-tier-flask-app:latest
                    '''
                }
            }
        }

        stage("Deploy") {
            steps {
                sh '''
                    docker compose down || true
                    docker rm -f mysql || true
                    docker compose up -d
                '''
            }
        }
    }

    post {
        success {
            mail(
                to: "gopesh7710@gmail.com",
                subject: "Build Successful",
                body: "Good: Your Build Was Successful!"
            )
        }
        failure {
            mail(
                to: "gopesh7710@gmail.com",
                subject: "Build Failed",
                body: "Bad: Your Build Failed!"
            )
        }
    }
}
