pipeline {
     
  agent any 
  environment {
    DOCKERHUB_CREDENTIALS=credentials('dockerhub') // Create a credentials in jenkins using your dockerhub username and token from https://hub.docker.com/settings/security
  }
  
  stages {

    stage("Git Checkout") {
      steps {
        container ("default") {
           sh "git clone https://github.com/abobakrahmed/cicd-java-maven-project.git"            
        }
      }
    }

    stage("Maven Build") {
      steps {   
        container ("default") {
          sh "mvn clean install -T 1C" // -T 1C is to make build faster using multithreading
        }
      }
    }

    stage("Run SonarQube Analysis") {
       steps {
           container ("jnlp") {
           withSonarQubeEnv('Sonarqube') {
            sh 'mvn clean package sonar:sonar -Dsonar.profile="Sonar way -Dsonar.host.url=http://3.127.136.150:9000 --Dsonar.projectKey=cicd-maven-staging"'
                }
              }
           }
    }
       
//     stage("Quality Gate") {
//        steps {  
//           timeout(time: 10, unit: 'MINUTES') {
//                   waitForQualityGate abortPipeline: true
//                }
//           }
//      }    

    stage("Build & Push Docker Image") {
      steps {
        container ("jnlp") {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh "docker build -t abobakr/cicd-java-maven ."
          sh "docker push abobakr/cicd-java-maven"
        }
      }
    }

    stage("Apply the Kubernetes files") {
      steps {
        container ("jnlp") {
          sh "kubectl apply -f kubernetes/ "
        }
      }
    }
  }
}
