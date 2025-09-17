pipeline {
  agent { docker { image 'maven:3.8.8-openjdk-11' args '-v /root/.m2:/root/.m2' } }
  environment {
    PROJECT_DIR = "app"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build & Unit Tests') {
      steps {
        sh "cd ${PROJECT_DIR} && mvn -B -DskipTests=false clean verify"
      }
    }
    stage('SonarQube Scan') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          withSonarQubeEnv('SonarQubeServer') {
            sh "cd ${PROJECT_DIR} && mvn -B sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
          }
        }
      }
    }
    stage('Quality Gate') {
      steps {
        timeout(time: 3, unit: 'MINUTES') {
          script {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
              error \"Quality Gate failed: ${qg.status}\"
            }
          }
        }
      }
    }
    stage('Package & Artifact') {
      steps {
        sh "cd ${PROJECT_DIR} && mvn -B -DskipTests package"
        archiveArtifacts artifacts: "${PROJECT_DIR}/target/*.jar", fingerprint: true
      }
    }
  }
  post {
    always { echo "Pipeline finished" }
  }
}
