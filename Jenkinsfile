pipeline {
    agent any
    
    environment {
        registry = "529088272063.dkr.ecr.eu-north-1.amazonaws.com/vanakkam-jenkins-repo"
    }

    tools {
        // Assuming Maven is installed globally in Jenkins
        maven 'mymaven' // Change this to the name of your installed Maven tool in Jenkins
    }
   
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Meenakshi0812/jenkins-ECR.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                // Run Maven clean and install to build the project
                sh 'mvn clean install'
            }
        }

        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws_cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 529088272063.dkr.ecr.eu-north-1.amazonaws.com'
                        sh 'docker push 529088272063.dkr.ecr.eu-north-1.amazonaws.com/vanakkam-jenkins-repo:latest'
                    }
                }
            }
        }

        stage('Stop previous containers') {
            steps {
                sh 'docker ps -f name=mytomcatContainer -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -fname=mytomcatContainer -q | xargs -r docker container rm'
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    sh 'docker run -d -p 9096:8080 --rm --name mytomcatContainer 529088272063.dkr.ecr.eu-north-1.amazonaws.com/vanakkam-jenkins-repo:latest'
                }
            }
        }
    }
}
