def notify(message) {
    def token = "eZxo5wAFVPNl33BKaZr3XtfvojxJ0dZyIKnARTQB3UZ";
    def jobName = env.JOB_NAME + ' - ' + env.BRANCH_NAME;
    def buildNo = env.BUILD_NUMBER;
      
    def url = "https://notify-api.line.me/api/notify";
    def lineMessage = "${jobName} [#${buildNo}] : ${message} \r\n";
    sh "curl ${url} -H 'Authorization: Bearer ${token}' -F 'message=${lineMessage}'";
}

pipeline {
    agent any
    environment {
        PROJECT_ID = 'yaprompt-dev-372404'
        CLUSTER_NAME = 'yaprompt-dev'
        LOCATION = 'asia-southeast1-a'
        CREDENTIALS_ID = 'Yaprompt-dev'
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
                notify("Jenkins start checkout code");
            }
        }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("witchayut/nestjs-k8s:${env.BUILD_ID}")
                    notify("Jenkins build docker");
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerID') {
                        myapp.push("latest")
                        myapp.push("${env.BUILD_ID}")
                        notify("Jenkins docker build has been pushed");
                    }
                }
            }
        }        
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    }    
    post {
        success {
            notifyLINE("succeed")
        }
        failure {
            notifyLINE("failed")
        }
    }
}