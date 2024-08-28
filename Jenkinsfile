pipeline {

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    } 
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

   agent  any
     tools{
        jdk 'openJDK8'
        maven 'maven3'
        
    }
   
    stages {
         stage('SCM') {
            steps {
               git changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/docker-spring-boot-java-web-service-example.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean install'
            }
        }
          stage('Docker Build and Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerhub2') {
                        sh 'docker build -t devopsike123/devospike:tag123 .'
                        sh 'docker push devopsike123/devospike:tag123 '
    
                    }
                    
                }
                
            }
        }

        stage('checkout') {
            steps {
                 script{
                        dir("terraform")
                        {
                            git "https://github.com/kingofdarknesss/docker-terraform-jenkins.git"
                        }
                    }
                }
        }

        stage('Plan') {
            steps {
                sh 'pwd;cd terraform/ ; terraform init'
                sh "pwd;cd terraform/ ; terraform plan -out tfplan"
                sh 'pwd;cd terraform/ ; terraform show -no-color tfplan > tfplan.txt'
            }
        }
        stage('Approval') {
           when {
               not {
                   equals expected: true, actual: params.autoApprove
               }
           }

           steps {
               script {
                    def plan = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            steps {
                sh "pwd;cd terraform/ ; terraform apply -input=false tfplan"
            }
        }
    }

  }