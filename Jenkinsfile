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
    image: jenkins/jnlp-agent-maven:jdk11
    imagePullPolicy: Always
    command: ["sleep", "100000"] 
    securityContext:
       allowPrivilegeEscalation: false
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command: ["sleep"]
    args: ["200"]
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: kaniko-secret
    secret:
        secretName: reg-credentials
        items:
        - key: .dockerconfigjson
          path: config.json     
'''
        }
  }
  environment {
    DOCKERHUB_CREDENTIALS=credentials('dockerhub') // Create a credentials in jenkins using your dockerhub username and token from https://hub.docker.com/settings/security
    KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'   
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
           withSonarQubeEnv('Sonarqube') {
            sh 'mvn clean package sonar:sonar -Dsonar.profile="Sonar way -Dsonar.host.url=http://3.127.136.150:9000 --Dsonar.projectKey=cicd-maven-staging"'
                }
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
        container ('kaniko') {
          sh '''
            /kaniko/executor --context `pwd` --destination=abobakrahmed1/cicd-maven-app:1.0.0
        ''' 
        }
      }
    }

    stage("Apply the Kubernetes files") {
      steps {
        container ('maven') {   
           withKubeCredentials(kubectlCredentials: [[caCertificate: '', 
               clusterName: 'arn:aws:eks:ca-central-1:749338631975:cluster/eksPoc', contextName: '', credentialsId: 'TestKubernetes', namespace: 'kube-system', 
               serverUrl: 'https://5E95D6E384E16CA6F220DE5F93348A36.yl4.ca-central-1.eks.amazonaws.com']]) {
                    sh "su - root" 
                    sh "curl -L https://storage.googleapis.com/kubernetes-release/release/v1.23.6/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl"
                    sh "chmod +x /usr/local/bin/kubectl"   
                    sh "envsubst < kubernetes/* | kubectl apply -f - "
             }
        }
      }
    }
  }
}

