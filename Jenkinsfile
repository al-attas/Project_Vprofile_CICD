pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }


    environment {
        registryCredential = 'ecr:us-west-2:awscreds'
        appRegistry = "017540984032.dkr.ecr.us-west-2.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://017540984032.dkr.ecr.us-west-2.amazonaws.com"
    }
  stages {
   
        stage('Fetch code') {
            steps {
               git branch: 'docker', url: 'https://github.com/al-attas/Project_Vprofile_CICD.git'
            }

        }


        stage('Build'){
            steps{
               sh 'mvn install -DskipTests'
            }

            post {
               success {
                  echo 'Now Archiving it...'
                  archiveArtifacts artifacts: '**/target/*.war'
                 }
            }
        }

        stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage("Sonar Code Analysis") {
            environment {
                scannerHome = tool 'sonar8.0'
            }
            steps {
              withSonarQubeEnv('sonarserver') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Project-CICD-TEST \
                   -Dsonar.projectName=Project-CICD-TEST \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build App Image') {
          steps {
       
            script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
          }
    
        }

        stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }
	
	stage("remove container images"){
	    steps{
	    	sh 'docker rmi -f $(docker images -a -q)'
	    }
	}
  }
}