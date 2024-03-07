pipeline {
    agent any
    
    
      tools {
        jdk 'java17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'                      
        
          /// THIS IS FOR DOCKER CRED TO PUSH 
        APP_NAME = "complete-prodcution-e2e-pipeline"      
        RELEASE = "1.0.0"
        DOCKER_USER = "raemondarellano"
        DOCKER_PASS = 'jenkins-docker-credentials'              
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"  
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        // add jenkins token file 
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

     stages{
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/iam-arellano/complete-prodcution-e2e-pipeline.git'
            }

        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }
        
       stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube_access') {
                    sh "mvn sonar:sonar"
                     }
                }
            }
        }    
        
      

   stage("Quality gate"){
            steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube_access'
                }

            }
        }
        
           stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

        }
        // to trigger other pipeline to update jenkins manifest
         stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://192.168.100.74:8080/job/gitops-complete-prodcution-e2e-pipeline/buildWithParameters?token=gitops-token-e2e'"
                }
            }
        }
    }
}
