pipeline {
    
    agent {
        label "build"
    }
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/kenchedda/CICD-EKS-NEXUS.git'
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
            stage('nexus_uploader') {
                steps{
                    script {
                        def readPomVersion = readMavenPom file: 'pom.xml'
                        def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "mavensnap" : "mavenrepo"

                        nexusArtifactUploader artifacts:
                         [
                            [
                                artifactId: 'springboot',
                                classifier: '',
                                file: 'target/Uber.jar',
                                type: 'jar'
                                ]
                            ], 
                                credentialsId: 'nexus_cred', 
                                groupId: 'com.example', 
                                nexusUrl: '18.188.81.130:8081', 
                                nexusVersion: 'nexus3', 
                                protocol: 'http', 
                                repository: nexusRepo, 
                                version: "${readPomVersion.version}"

                    } 
        }
     
       }

                        stage('Docker image build'){
                    steps {
                        script {
                            sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                            sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kenappiah/$JOB_NAME:$BUILD_ID'
                            sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kenappiah/$JOB_NAME:latest1'
                        }
                    }
            
                }
            stage ('publish docker image') {
                steps{
                    script{
                        withCredentials([string(credentialsId: 'dockersec', variable: 'docker_hub_cred')]) {
                            sh 'docker login -u kenappiah -p ${docker_hub_cred}'
                            sh 'docker image push kenappiah/$JOB_NAME:latest1'
                    }
                }
            }                
        }  
                stage(" Deploy ") {
       steps {
         script {
            echo '<--------------- Helm Deploy Started --------------->'
            
        
            sh 'helm install alina /home/ubuntu/alina-0.1.0.tgz'
        
            echo '<--------------- Helm deploy Ends --------------->'
         }
       }
     }  
}
}
