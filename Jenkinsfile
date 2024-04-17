pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('Dockerhub')
    }
    stages {
        stage('Verify Branch') {
            steps {
                echo "$GIT_BRANCH"
            }
        }
        stage('Login to Dockerhub') {
            steps {
                script {
                    // Retrieve Dockerhub credentials from Jenkins
                    withCredentials([usernamePassword(credentialsId: 'Dockerhub', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW', usernameVariable: 'DOCKERHUB_CREDENTIALS_USR')]) {
                        // Log in to Dockerhub
                        bat 'docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}'
                    }
                }
            }
        }
        stage('Pulling base image from Dockerhub') {
            steps {
                bat 'docker pull abodiaa/ausf-base:latest'
            }
        }
        stage('docker build') {
            steps {
                bat(script: """
                    docker images -a
                    docker build -t abodiaa/ausf:latest . 
                    docker images -a
                """)
            }
        }
        stage('Scan Image for Common Vulnerabilities and Exposures') {
            steps {
                bat 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v C:/Users/BADR/AppData/Local/Docker/wsl:/images aquasec/trivy image abodiaa/ausf:latest --output /images/trivy-report.json'
            }
        }

        stage('Pushing to Dockerhub') {
            steps {
                bat 'docker push abodiaa/ausf:latest'
            }
        }
        stage('Build and Package Helm Chart') {
            steps {
                bat 'helm package ./helm/'
            }
        }
        stage('Configure Kubernetes Context') {
            steps {
                bat 'aws eks --region us-east-1 update-kubeconfig --name 5G-Core-Net'
            }
        }
        stage('Deploy Helm Chart on EKS') {
            steps {
                bat 'helm upgrade --install amf ./helm/'
            }
        }
    }
    post {
        always {
            // Archiving Test Result
            archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
            bat 'docker rmi abodiaa/ausf:latest'
            bat 'docker rmi abodiaa/ausf-base:latest'
        }
    }
}