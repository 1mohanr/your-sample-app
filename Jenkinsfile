pipeline {
    agent any

    environment {
        DOCKERHUB_CRED = credentials('dockerhub-username')   // Jenkins DockerHub username credential ID
        DOCKERHUB_CRED_PSW = credentials('dockerhub-password') // Jenkins DockerHub password credential ID
        SONAR_TOKEN = credentials('sqa_b542c6bae4af53dd989a805d47c626ee3cdb364a') // SonarQube token
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Mount a writable directory for scanner cache
                    docker.image('sonarsource/sonar-scanner-cli:latest').inside("-v ${env.WORKSPACE}/.sonar:/opt/sonar-scanner/.sonar") {
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
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Build Artifact') {
            steps {
                sh '''
                # Assuming Maven project
                mvn clean package
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-cred') {
                        def appImage = docker.build("ramm978/your-sample-app:${env.BRANCH_NAME}")
                        appImage.push()
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

