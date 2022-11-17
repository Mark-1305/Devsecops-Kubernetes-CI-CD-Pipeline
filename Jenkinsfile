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
      stage('SonarQube - SAST') {
      steps {
        sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://10.0.8.45:9000 -Dsonar.login=f4e63a0ba58466b6caae9dbebb94f6c4e5886721"
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
