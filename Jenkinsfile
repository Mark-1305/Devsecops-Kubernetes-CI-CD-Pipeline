pipeline {
  agent any
    environment  {
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
        IMAGE_NAME = "manoharshetty507/devsecops-numeric-app"

    }  
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }   
      stage('Unit Tests - Junit and Jacoco') {
            steps {
              sh "mvn test"
            }     
           }  
        stage('Mutation Tests - PIT') {
          steps {
            sh "mvn org.pitest:pitest-maven:mutationCoverage"
           }
        }
        stage('Static Code Analysis'){
            steps{
                script{
                withSonarQubeEnv(credentialsId: 'sonar-token') {
                    sh "mvn clean package sonar:sonar"
                    }                    
                }
            }
        } 
        stage('Quality Gate Status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                } 
            }
        }
       stage('Vulnerability Scan - Docker') {
          steps {
          		sh "mvn dependency-check:check"
          }
       }
      stage('Docker build and push'){
           steps{
            withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            sh 'sudo docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            sh 'docker push $IMAGE_NAME:$IMAGE_TAG'  
          }
        }
      }
    stage('Kubernetes Deployment - Jenkins-Pipeline'){
          steps{
            withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#manoharshetty507/devsecops-numeric-app:$IMAGE_TAG#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml "
            }
        }
    }
  }
}
post { 
    always {
        junit 'target/surefire-reports/*.xml'
        jacoco execPattern: 'target/jacoco.exec' 
        pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'

    }
}
