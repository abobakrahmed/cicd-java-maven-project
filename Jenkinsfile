pipeline {
     
  agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: maven-staging
  namespace: jenkins-new
spec:
  securityContext:
    runAsUser: 0
  containers:
  - name: maven
    image: jenkins/jnlp-agent-maven:latest
    command: ["sleep", "100000"]
    securityContext:
      allowPrivilegeEscalation: false
'''
        }
  }
//   agent {
//       kubernetes {
//           inheritFrom 'maven'
//       }
//   }
  environment {
    DOCKERHUB_CREDENTIALS=credentials('dockerhub') // Create a credentials in jenkins using your dockerhub username and token from https://hub.docker.com/settings/security
  }
  
  stages {

    stage("Git Checkout") {
      steps {
        script {
           sh "git clone https://github.com/abobakrahmed/cicd-java-maven-project.git"            
        }
      }
    }

    stage("Maven Build") {
      steps {   
        container ('maven') {
          sh "mvn clean install -T 1C" // -T 1C is to make build faster using multithreading
        }
      }
    }

   stage("Run SonarQube Analysis") {
      steps {
        container ('maven') {
          script {
          withSonarQubeEnv(credentialsId: 'Sonarqube', installationName: 'Sonarqube') {
           sh 'mvn clean package sonar:sonar -Dsonar.profile="Sonar way"'
          }
          try {
            timeout(time: 5, unit: 'MINUTES') { // pipeline will be killed after a timeout
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
            }
          } catch (e) {
            throw e
          }
        }
      }
    }
  }

    stage("Build & Push Docker Image") {
      steps {
        script {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh "docker build -t abobakr/cicd-java-maven ."
          sh "docker push abobakr/cicd-java-maven"
        }
      }
    }

    stage("Apply the Kubernetes files") {
      steps {
        script {
          sh "kubectl apply -f kubernetes/ "
        }
      }
    }
  }
//   post {
//     always {
//       script {
//         if (currentBuild.currentResult == 'FAILURE') {
//           step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: "bakkorahmed5@gmail.com", sendToIndividuals: true])
//         }
//       }
//     }
//   }
}
