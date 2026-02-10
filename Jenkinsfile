pipeline{
    agent {label "dev"} ;
    stages{
        stage("Code Clone"){
            steps{
                git url : "https://github.com/gopeshthakare13/two-tier-flask-app.git"
            }
        }
         stage("Installation Command To Docker Build"){
            steps{
                sh '''
                  sudo apt update
                  sudo apt install -y docker.io docker-compose-v2
                  sudo usermod -aG docker ubuntu
                  sudo usermod -aG docker $USER
                  newgrp docker
                   '''
            }
        }
        stage("Code Build"){
            steps{
                sh "docker build -t two-tier-flask-app ."
            }
        }
        stage("Test Case"){
            steps{
                echo "Developer Write Test Cases Here ...."
            }
        }
       stage("Docker Push") {
             steps {
                 withCredentials([usernamePassword(
                 credentialsId: "dockerHubCreds",
                 passwordVariable: "DOCKER_PASS",
                 usernameVariable: "DOCKER_USER"
               )]) {
                 sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag two-tier-flask-app $DOCKER_USER/two-tier-flask-app:latest
                    docker push $DOCKER_USER/two-tier-flask-app:latest
                    '''
                }
           }
       }
        stage("Deploy"){
            steps{
                sh "docker compose up -d"
            }
        }
    }
}
