pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/RAMESHKUMARVERMAGITHUB/demo_nodejs_webpage.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=demo_nodejs_webpage \
                    -Dsonar.projectKey=demo_nodejs_webpage'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
        }
        stage("Docker Build & Push"){
             steps{
                 script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                      sh "docker build -t rameshkumarverma/demo_nodejs_webpage:latest ."
                    //   sh "docker tag demo_nodejs_webpage rameshkumarverma/demo_nodejs_webpage:latest "
                      sh "docker push rameshkumarverma/demo_nodejs_webpage:latest "
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image rameshkumarverma/demo_nodejs_webpage:latest > trivyimage.txt" 
            }
        }
        stage('docker deploy'){
            steps{
                sh "docker run -d -p 8080:8080 rameshkumarverma/demo_nodejs_webpage:latest" 
            }
        }
        stage('Deploy to Kubernets'){
            steps{
                script{
                    // dir('kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        // sh 'kubectl delete --all pods'
                        // sh 'kubectl apply -f deployment.yml'
                        // sh 'kubectl apply -f service.yml'
                          sh 'kubectl apply -f k8s.yml'
                        }   
                    // }
                }
            }
        }
    }
    // post {
    //  always {
    //     emailext attachLog: true,
    //         subject: "'${currentBuild.result}'",
    //         body: "Project: ${env.JOB_NAME}<br/>" +
    //             "Build Number: ${env.BUILD_NUMBER}<br/>" +
    //             "URL: ${env.BUILD_URL}<br/>",
    //         to: 'rootmeet@gmail.com',                              
    //         attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    //     }
    // }
}
