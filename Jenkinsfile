pipeline {
    
    agent {
        label "build"
    }


    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/kenchedda/ECR.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
         stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
          stage('Static code analysis'){
            
            steps{
                
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonar') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
            stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                    }
                }
            }

             stage('Docker image build'){
                    steps {
                        script {
                            sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                            sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kenappiah/$JOB_NAME:latest'
                        }
                    }
            
                }
            stage('Docker image scan'){
                    steps {
                        script {
                            sh 'trivy image $JOB_NAME:v1.$BUILD_ID '
                            sh 'trivy image  kenappiah/$JOB_NAME:latest'
                        }
                    }
            
                }

            stage ('publish docker image') {
                steps{
                    script{
                        withCredentials([string(credentialsId: 'dockersec', variable: 'docker_hub_cred')]) {
                            sh 'docker login -u kenappiah -p ${docker_hub_cred}'
                            sh 'docker image push kenappiah/$JOB_NAME:latest'
                    }
                }
            }                
        }

            stage('Create EKS Cluster : Terraform'){
                steps{
                  script{

                  
                      sh """
                          
                          terraform init 
                          terraform apply --auto-approve
                      """
                  
                }
            }
        }

    stage('Connect to EKS '){
            
        steps{

            script{

                sh """
                
                
                aws eks --region=us-east-1 update-kubeconfig --name=ed-eks-01
                """
            }
        }
        }  

    stage('deploy '){
            
        steps{

            script{

                sh """
                curl -LO https://dl.k8s.io/release/v1.27.0/bin/linux/amd64/kubectl
                chmod a+x kubectl
                mv kubectl /usr/local/bin
                kubectl apply -f .
                """
            }
        }
        }  
    }

}    