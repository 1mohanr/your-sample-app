pipeline {
    agent any

    environment {
        DOCKERHUB_CRED = credentials('dockerhub-ramm978')   // Your DockerHub credential ID
        SONAR_TOKEN = credentials('sqa_b542c6bae4af53dd989a805d47c626ee3cdb364a')  // SonarQube token ID
        SONAR_CACHE = "${env.WORKSPACE}/.sonar-cache"
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
                    // Ensure the cache directory exists
                    sh "mkdir -p ${SONAR_CACHE}"

                    // Use Dockerized Sonar Scanner
                    docker.image('sonarsource/sonar-scanner-cli:latest').inside('-u $(id -u):$(id -g) -v "${SONAR_CACHE}:/opt/sonar-scanner/.sonar"') {
                        sh """
                            sonar-scanner \
                                -Dsonar.projectKey=mohan.developer \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://3.107.198.86:9000 \
                                -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-ramm978') {
                        def appImage = docker.build("ramm978/mohan-app:${env.BUILD_NUMBER}")
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

