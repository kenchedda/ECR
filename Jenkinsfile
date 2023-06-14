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
                            sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/b4z2b0e5'
                            sh 'docker build -t myecr55 .'
                            sh 'docker tag myecr55:latest public.ecr.aws/b4z2b0e5/myecr55:latest'
                            sh 'docker push public.ecr.aws/b4z2b0e5/myecr55:latest'
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
                sudo mv kubectl /usr/local/bin
                kubectl apply -f .
                """
            }
        }
        } 

          stage('destroy EKS Cluster : Terraform'){
                steps{
                  script{

                  
                      sh """
                          
                         
                          terraform destroy --auto-approve
                      """
                  
                }
            }
        } 
        }
    }
    