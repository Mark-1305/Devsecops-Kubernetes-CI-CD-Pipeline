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
  }
}
