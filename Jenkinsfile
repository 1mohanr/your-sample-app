pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sqa_b542c6bae4af53dd989a805d47c626ee3cdb364a')
        DOCKERHUB_CRED = credentials('dockerhub-ramm978') // DockerHub credentials ID
        SONAR_CACHE = "${WORKSPACE}/.sonar" // local cache directory
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
                    sh "mkdir -p $SONAR_CACHE" // ensure cache directory exists
                    // Use Dockerized Sonar Scanner with cache volume and same UID/GID as Jenkins user
                    docker.image('sonarsource/sonar-scanner-cli:latest').inside("-u $(id -u):$(id -g) -v $SONAR_CACHE:/opt/sonar-scanner/.sonar") {
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
                sh 'echo "Add your build commands here (e.g., mvn package)"'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-ramm978') {
                        sh 'docker build -t ramm978/your-app:latest .'
                        sh 'docker push ramm978/your-app:latest'
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
            echo 'Pipeline failed. Check logs for errors.'
        }
    }
}

