pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials-id'
        SONARQUBE_URL = 'http://sonarqube.sonarqube:9000'
        NEXUS_URL = 'http://nexus.nexus:8081'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/dineshsai07/kubernetes-java.git'
            }
        }
        stage('Set Permissions') {
            steps {
                sh 'chmod +x mvnw'
            }
        }
        stage('Lint') {
            steps {
                script {
                    sh './mvnw checkstyle:check'
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    sh './mvnw clean package'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    sh './mvnw test'
                }
                // Archive test results
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Code Coverage') {
            steps {
                script {
                    sh './mvnw jacoco:report'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh './mvnw sonar:sonar'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('your_docker_hub_username/demo:latest')
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy image your_docker_hub_username/demo:latest'
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS) {
                        sh 'docker push your_docker_hub_username/demo:latest'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            junit 'target/surefire-reports/*.xml'
            publishHTML(target: [
                reportName: 'Code Coverage',
                reportDir: 'target/site/jacoco',
                reportFiles: 'index.html'
            ])
        }
    }
}
