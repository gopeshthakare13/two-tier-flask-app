pipeline {
    agent { label "dev" }

    tools {
        dependencyCheck 'OWASP-DC'
    }

    stages {

        stage("Code Clone") {
            steps {
                git url: "https://github.com/gopeshthakare13/two-tier-flask-app.git"
            }
        }

        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs . --format json -o results.json"
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'OWASP-DC'
            }
        }

        stage("Code Build") {
            steps {
                sh "docker build -t two-tier-flask-app ."
            }
        }

        stage("Test Case") {
            steps {
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

        stage("Deploy") {
            steps {
                sh "docker compose down || true"
                sh "docker rm -f mysql || true"
                sh "docker compose up -d"
            }
        }
    }

    post {

        always {
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        }

        success {
            mail(
                to: "gopesh7710@gmail.com",
                subject: "Build Successful - ${env.JOB_NAME}",
                body: "Good: Your Build Was Successful!\nBuild URL: ${env.BUILD_URL}"
            )
        }

        failure {
            mail(
                to: "gopesh7710@gmail.com",
                subject: "Build Failed - ${env.JOB_NAME}",
                body: "Bad: Your Build Failed!\nCheck: ${env.BUILD_URL}"
            )
        }
    }
}

