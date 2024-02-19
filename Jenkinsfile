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
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://40.71.216.57:9000 -Dsonar.login=sqp_f5d8e786ab6a7be374fee768bb3722348ea93102"
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
