pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws_access_key')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
    }
    stages {
        stage('Deploy infrastructure') {
            steps {
                sh 'terraform init'
                sh 'terraform apply -auto-approve'
            }
        }
        stage('Build and deploy app') {
            steps {
                git 'https://github.com/boxfuse/boxfuse-sample-java-war-hello.git'
                sh 'mvn package'
                sh 'docker build -t hello-app .'
                sh 'docker tag hello-app:latest registry.example.com/hello-app:v1.0.' + env.BUILD_NUMBER
                withCredentials([usernamePassword(credentialsId: 'registry_cred', usernameVariable: 'REGISTRY_USERNAME', passwordVariable: 'REGISTRY_PASSWORD')]) {
                    sh 'docker login -u $REGISTRY_USERNAME -p $REGISTRY_PASSWORD registry.example.com'
                }
                sh 'docker push registry.example.com/hello-app:v1.0.' + env.BUILD_NUMBER
                // deploy container to staging
            }
        }
    }
    post {
        always {
            sh 'terraform destroy -auto-approve'
        }
    }
}
