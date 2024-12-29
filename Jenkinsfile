pipeline {
    agent any
    
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-repo', url: 'https://github.com/Omprakash-26/Boardgame.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                 sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardgame -Dsonar.projectKey=Boardgame \
                        -Dsonar.java.binaries=.'''
                  }
            }
        }
        /*stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-server'
                }
            }
        }*/
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh "mvn deploy"
               }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerhub-cred') {
                      sh "docker build -t omprakash194/secondimage:latest ."
                  }
                }
               
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html omprakash194/secondimage:latest"
            }
        }
        stage('Push Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerhub-cred') {
                      sh "docker push omprakash194/secondimage:latest"
                  }
                }
               
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'minikube', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.59.107:8443') {
                   sh 'kubectl apply -f deploy-service.yml'
                }
            }
        }
        stage('Verify The Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'minikube', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.59.107:8443') {
                   sh 'kubectl get pods -n webapps'
                   sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}
