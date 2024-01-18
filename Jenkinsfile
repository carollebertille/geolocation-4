pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
    environment {
        registry = '792069373652.dkr.ecr.us-east-2.amazonaws.com/carolle'
        dockerimage = '' 
    }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/Hermann90/geolocation.git'
            }
        }
        stage('Code test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Code build') {
            steps {
                sh 'mvn clean package'
            }
        }
        // Building Docker images
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    // Authenticate with AWS ECR
                    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential) { 
                      dockerImage.push()
                  }
            }
          }
        }
        //deploy the image that is in ECR to our EKS cluster
        stage ("Kube Deploy") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'eks_credential', namespace: '', serverUrl: '') {
                 sh "kubectl apply -f eks-deploy-from-ecr.yaml"
                }
            }
        }
    }
}
