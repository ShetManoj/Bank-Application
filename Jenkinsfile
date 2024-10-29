pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SONAR_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ShetManoj/Bank-Application.git'
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
        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=BankApplication -Dsonar.projectKey=BankApplication -Dsonar.java.binaries=target"
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'publish-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                   sh 'mvn deploy'
                }
            }
        }
        stage('Build Image and Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker build -t 1manojshet/bankapplication:kind-1 ."
                    }
                    
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o image.html 1manojshet/bankapplication:kind-1'
            }
        }
        stage('Push Image to Registry') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker push 1manojshet/bankapplication:kind-1"
                    }
                    
                }
            }
        }
        stage('Deployment to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' manoj-cluster', contextName: '', credentialsId: 'kubernetes-cred', namespace: 'manoj', restrictKubeConfigAccess: false, serverUrl: 'https://30E899A7DA790FA0C319C2EA993BA7B3.gr7.us-east-1.eks.amazonaws.com') {
                   sh "kubectl apply -f ds.yml -n manoj"
                   sleep 30
                }
            }
        }
        stage('Deployment Verification') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' manoj-cluster', contextName: '', credentialsId: 'kubernetes-cred', namespace: 'manoj', restrictKubeConfigAccess: false, serverUrl: 'https://30E899A7DA790FA0C319C2EA993BA7B3.gr7.us-east-1.eks.amazonaws.com') {
                   sh "kubectl get pods -n manoj"
                   sh "kubectl get svc -n manoj"
                }
            }
        }
         
    }
}
