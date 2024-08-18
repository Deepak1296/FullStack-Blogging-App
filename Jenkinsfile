pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Deepak1296/FullStack-Blogging-App.git'
            }
        }
        
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        
        stage('Trivy Fs Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app -Dsonar.projectKey=Blogging-app \
                -Dsonar.java.binaries=target'''
                }
            }
        }
        
        
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                   sh "mvn deploy"
                }
            }
        }
        
        
        stage('Docker Build and Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                       sh "docker build -t deepak121099/bloggingapp:latest ."
                    }
                }
            }
        }
        
        
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html deepak121099/bloggingapp:latest"
            }
        }
        
        
        stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                       sh "docker push deepak121099/bloggingapp:latest"
                    }
                }
            }
        }
        
        stage('K8-Deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://65EFAAFDA77F5377B9E0868881E2670B.gr7.ap-south-1.eks.amazonaws.com') {
                sh "kubectl apply -f deployment-service.yml"
                sleep 20
                }
            }
        }
        
        
        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://65EFAAFDA77F5377B9E0868881E2670B.gr7.ap-south-1.eks.amazonaws.com') {
                sh "kubectl get pods"
                sh "kubectl get svc"
            
                }
            }
        }
        
        
        
    }
}
