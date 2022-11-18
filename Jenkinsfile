pipeline {
  agent any
  stages {
      stage('Clone repository') {
      current = "Stage Cloning"
      checkout scm
      GIT_BRANCH = gitscm.GIT_BRANCH.replace("/", "");
      GIT_BRANCH=GIT_BRANCH.split("origin")
    }
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
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                } 
            }
        }
       stage('Vulnerability Scan - Docker') {
          steps {
          	sh "mvn dependency-check:check"
          }
        post {
          always{
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
          }
        }
      }
      stage('Docker build and push'){
           steps{
            withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            sh 'sudo docker build -t manoharshetty507/devsecops-numeric-app:""v1.$BUILD_ID"" .'
            sh 'docker push manoharshetty507/devsecops-numeric-app:""v1.$BUILD_ID""'  
          }
        }
      }
    }   
}
