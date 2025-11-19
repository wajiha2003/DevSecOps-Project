pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_REGISTRY = "wajihaaz2003" // change to your Docker Hub username
        IMAGE_NAME = "netflix-app"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix"""
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '--format HTML --out dependency-check-report.html'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:latest")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest'
            }
        }

        stage('Deploy to Container') {
            steps {
                sh 'docker run -d -p 8080:8080 ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest'
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
            archiveArtifacts artifacts: '**/dependency-check-report.html', allowEmptyArchive: true
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
