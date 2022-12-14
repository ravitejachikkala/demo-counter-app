pipeline{
    
    agent any 
    environment {
        PATH = "/opt/maven/bin/:$PATH"
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-nexus')
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
                        
                        def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "nexus-snapshot" : "nexus-release"
                        
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
                            nexusUrl: '3.108.193.108:8081',
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            repository: nexusRepo,
                            //version: '1.0.0'
                            version: "${readPomVersion.version}"
                    }
                }
    
            }
            stage('Build Docker Image'){
                steps{
                    script{ 
                        sh 'docker build -t raviteja2/demo1:$BUILD_ID .'
                       // sh 'docker image tag $JOB_NAME:v1.$BUILD_ID raviteja2/$JOB_NAME:v1.$BUILD_ID'
                       // sh 'docker image tag $JOB_NAME:v1.$BUILD_ID raviteja2/$JOB_NAME:latest'
                    }
                }
                
            }
        
        stage('Push Docker image') {
            steps{
               sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
               sh 'docker push raviteja2/demo1:$BUILD_ID'
                
            }
        
        }
}
    
}
