pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sqa_b542c6bae4af53dd989a805d47c626ee3cdb364a') // your SonarQube secret
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
                    // Use official SonarScanner Docker image with a writable cache directory
                    docker.image('sonarsource/sonar-scanner-cli:latest').inside("-v ${env.WORKSPACE}/.scanner-cache:/opt/sonar-scanner/.sonar/cache") {
                        sh '''
                            sonar-scanner \
                                -Dsonar.projectKey=${JOB_NAME} \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://3.107.198.86:9000 \
                                -Dsonar.login=${SONAR_TOKEN}
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Quality Gate stage will run only if SonarQube succeeds."
                // Add sonar-quality-gate step if you have SonarQube plugin configured
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

