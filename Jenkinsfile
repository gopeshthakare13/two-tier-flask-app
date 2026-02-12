pipeline{
    agent {label "dev"} ;

     tools {
        dependencyCheck 'OWASP-DC'                                            /* OWASP :-  Your project uses third-party libraries (like Flask, requests, MySQL connector).
                                                                                            Some versions may have known security vulnerabilities. */

                                                                                          /*Solution:
                                                                                               Use OWASP Dependency-Check in Jenkins pipeline.*/
    }
    
    stages{
        stage("Code Clone"){
            steps{
                git url : "https://github.com/gopeshthakare13/two-tier-flask-app.git"
            }
        }
        
        stage("Trivy File System Scan){                                                /*Trivy Trivy is a security scanner for code, containers, Kubernetes, and IaC */
            steps{
               sh "trivy fs . --format json -o results.json"
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'OWASP-DC'
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
               sh "docker compose down || true"
               sh "docker rm -f mysql || true"
               sh "docker compose up -d"
            }
        }
    }
post {                           /* Email Notifiaction */

    always {
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        }
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
