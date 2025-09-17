pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sqa_b542c6bae4af53dd989a805d47c626ee3cdb364a')
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
                    docker.image('sonarsource/sonar-scanner-cli:latest').inside {
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
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
            cleanWs()
        }
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Pipeline succeeded!'
        }
    }
}

