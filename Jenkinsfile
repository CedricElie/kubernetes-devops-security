def stageResults = [:]
pipeline {
    agent any

    environment {
      deploymentName = "devsecops"
      containerName = "devsecops-container"
      serviceName = "devsecops-svc"
      imageName="cedricelie/numeric-app:${BUILD_ID}"
      applicationURL="http://devsecops.eastus.cloudapp.azure.com/"
      applicationURI="/increment/99"
    }
    
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
              withSonarQubeEnv('SonarQube') {
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://40.71.216.57:9000 -Dsonar.login=sqp_f5d8e786ab6a7be374fee768bb3722348ea93102"
              }
              timeout(time:2, unit: 'MINUTES') {
                script {
                  waitForQualityGate abortPipeline: true
                }
              }
            }
        }
        stage('Vulnerability Scan - Docker') {
          steps {
            parallel (
                "Dependency Scan":{
                    sh " mvn dependency-check:check"
                },
                "Trivy Scan":{
                    sh " bash trivy-docker-image-scan.sh"
                },
                "OPA Conftest":{
                  sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
                }

            )
          }
        }

        stage('Docker Build and Push') {
          steps {
            withDockerRegistry([credentialsId: "docker-hub",url: ""]) {
              sh 'printenv'
              sh 'sudo docker build -t cedricelie/numeric-app:""$BUILD_ID"" .'
              sh 'docker push cedricelie/numeric-app:""$BUILD_ID""'
            }
          }
        }
        stage('Vulnerability Scan - Kubernetes') {
          steps {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          }
        }
        // stage('Kubernetes Deployment - DEV') {
        //   steps {
        //     withKubeConfig([credentialsId: "kubeconfig"]) {
        //       sh 'printenv'
        //       sh "sed -i 's#latest#${BUILD_ID}#g' k8s_deployment_service.yaml"
        //       sh 'kubectl apply -f k8s_deployment_service.yaml'
        //     }
        //   }
        // }
        stage('K8s Deployment - DEV') {
          steps {
            parallel(
              "Deployment": {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                   sh "bash k8s-deployment.sh"
                }
              },
              "Rollout Status": {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                  sh "bash k8s-deployment-rollout-status.sh"
                }
              }
            )
          }
        }
    }
    post {
      always {
        junit 'target/surefire-reports/*.xml'
        jacoco execPattern: 'target/jacoco.exec'
        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        echo "Stage results: ${stageResults}"
      }
    }
}