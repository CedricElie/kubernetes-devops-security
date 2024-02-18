pipeline {
    agent any
    stages {
        stage('Build Artifact') { 
            steps {
                sh 'mvn clean package -DskipTest=true'
                archive 'target/*.jar'
            }
        }
        stage('Unit Tests') { 
            steps {
                sh 'mvn test'
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }
    }
}
