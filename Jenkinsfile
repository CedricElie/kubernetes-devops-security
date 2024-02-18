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
        stage('Docker Build and Push') { 
          steps {
            withDockerRegistry([credentialsId: "docker-hub",url: ""]) {
              sh 'printenv'
              sh 'docker build -t cedricelie/numeric-app:""$BUILD_ID"" .'
              sh 'docker push cedricelie/numeric-app:""$BUILD_ID""'
            }
          }
        }
        stage('Kubernetes Deployment - DEV') { 
          steps {
            withKubeConfig([credentialsId: "kubeconfig"]) {
              sh 'printenv'
              sh "sed -i 's#latest#${BUILD_ID}#g' k8s_deployment_service.yaml"
              sh 'kubectl apply -f k8s_deployment_service.yaml'
            }
          }
        }
    }
}
