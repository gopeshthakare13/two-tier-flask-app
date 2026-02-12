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
                sh """
                    trivy fs . \
                    --scanners vuln \
                    --severity HIGH,CRITICAL \
                    --exit-code 1
                """
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_KEY')]) {
                    dependencyCheck(
                        odcInstallation: 'OWASP',
                        additionalArguments: "--scan . --format XML --nvdApiKey ${NVD_KEY}"
                    )
                }

                dependencyCheckPublisher(
                    pattern: '**/dependency-check-report.xml',
                    failedTotalHigh: 1
                )
            }
        }

        stage("Docker Build") {
            steps {
                sh "docker build -t two-tier-flask-app:latest ."
            }
        }

        stage("Trivy Docker Image Scan") {
            steps {
                sh """
                    trivy image two-tier-flask-app:latest \
                    --severity HIGH,CRITICAL \
                    --exit-code 1
                """
            }
        }

        stage("Docker Push") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHubCreds",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker tag two-tier-flask-app:latest ${DOCKER_USER}/two-tier-flask-app:latest
                        docker push ${DOCKER_USER}/two-tier-flask-app:latest
                    """
                }
            }
        }

        stage("Deploy") {
            steps {
                sh """
                    docker compose down || true
                    docker compose up -d
                """
            }
        }
    }

    post {
        success {
            mail(
                to: "gopesh7710@gmail.com",
                subject: "Build Successful - ${env.JOB_NAME}",
                body: "Your build was successful.\nBuild URL: ${env.BUILD_URL}"
            )
        }

        failure {
            mail(
                to: "gopesh7710@gmail.com",
                subject: "Build Failed - ${env.JOB_NAME}",
                body: "Build failed due to errors or vulnerabilities.\nCheck: ${env.BUILD_URL}"
            )
        }
    }
}
