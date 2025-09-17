pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token') // SonarQube token in Jenkins
        DOCKERHUB_CRED = credentials('dockerhub-ramm978') // DockerHub credentials
        IMAGE_NAME = "ramm978/your-sample-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/1mohanr/your-sample-app.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    docker.image('sonarsource/sonar-scanner-cli:latest').inside {
                        sh """
                        sonar-scanner \
                        -Dsonar.projectKey=${env.BRANCH_NAME} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://3.107.198.86:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'mohan.developer'
            }
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${env.BRANCH_NAME} .
                    echo "${DOCKERHUB_CRED_PSW}" | docker login -u "${DOCKERHUB_CRED_USR}" --password-stdin
                    docker push ${IMAGE_NAME}:${env.BRANCH_NAME}
                    """
                }
            }
        }
    }
}

