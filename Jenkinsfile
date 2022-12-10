pipeline{
    
    agent any 
    environment {
        PATH = "/opt/maven/bin/:$PATH"
    }    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/ravitejachikkala/demo-counter-app.git'
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
                    
                    withSonarQubeEnv(credentialsId: 'sonar-api') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
            stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                        
                    }
                }
            }
            stage('upload war file to nexus'){
                steps{
                    script{
                        def readPomVersion = readMavenPom file: 'pom.xml'
                        
                        def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "Nexus-SNAPSHOT" : "Nexus-App-Release"
                        
                        nexusArtifactUploader artifacts:
                            [
                                [
                                    artifactId: 'springboot',
                                    classifier: '', file: 'target/Uber.jar',
                                    type: 'jar'
                                    ]
                            ],
                            credentialsId: 'nexus-auth',
                            groupId: 'com.example',
                            nexusUrl: '13.233.83.240:8081',
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            repository: nexusRepo,
                            version: "${readPomVersion.version}"
                    }
                }
    
            }
            stage('Build Docker Image'){
                steps{
                    script{ 
                        sh 'docker build -t $JOB_NAME/v1.$BUILD_ID .'
                        //sh 'docker image --tag $JOB_NAME:v1.$BUILD_ID raviteja2/$JOB_NAME:v1.$BUILD_ID'
                        //sh 'docker image --tag $JOB_NAME:v1.$BUILD_ID raviteja2/$JOB_NAME:latest'
                    }
                }
                
            }
}
    
}
