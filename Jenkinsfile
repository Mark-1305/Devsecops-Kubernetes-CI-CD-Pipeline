pipeline {
  agent any
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
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
           }  
        }  
        stage('Mutation Tests - PIT') {
          steps {
            sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
          post {
            always {
              pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
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
          parallel(
          	"Dependency Scan": {
          		sh "mvn dependency-check:check"
     		},
     		"Trivy Scan":{
     			sh "bash trivy-docker-image-scan.sh"
    		},
     		"OPA Conftest":{
     			sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
     		}   	
            )
          }
      }
      stage('Docker build and push'){
           steps{
            withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            sh 'sudo docker build -t manoharshetty507/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push manoharshetty507/numeric-app:""$GIT_COMMIT""'  
          }
        }
      }
    stage('Kubernetes Deployment - Dev'){
          steps{
            withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#manoharshetty507/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml "
        } 
      }   
    }
  }
}
