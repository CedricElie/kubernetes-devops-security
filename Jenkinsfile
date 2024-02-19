pipeline {
    agent any
    stages {
        stage('Build Artifact') {
            steps {
                sh 'mvn clean package -DskipTest=true'
                archive 'target/*.jar'
            }
        }
        stage('Unit Tests - Junit and Jacoco') {
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
        stage('Mutation Test - PIT') {
            steps {
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
              always {
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              }
            }
        }
        stage('SonarQube - SAST') {
            steps {
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://devsecops.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_667b9fc0adaa30a55a72a9a3fb2b97ca172138c0"
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
