pipeline {
    agent {
        label 'master'
    }
    environment {
        //be sure to replace "tominhhien1" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "tominhhien1/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'export JAVA_HOME=/var/lib/jenkins/tools/hudson.model.JDK && ./gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'ed865ac2-f8ef-489f-ab77-11a816a18c78') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECRED')]) {
                    sh 'mkdir -p ~/.kube'
                    sh 'cat $KUBECRED > ~/.kube/config'
                    sh 'envsubst < train-schedule-kube-canary.yml > train-schedule-kube-canary-out.yml'
                    sh 'kubectl apply -f train-schedule-kube-canary-out.yml'
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECRED')]) {
                    sh 'mkdir -p ~/.kube'
                    sh 'cat $KUBECRED > ~/.kube/config'
                    sh 'envsubst < train-schedule-kube-canary.yml > train-schedule-kube-canary-out.yml'
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                    sh 'envsubst < train-schedule-kube.yml > train-schedule-kube-out.yml'
                    sh 'kubectl apply -f train-schedule-kube-out.yml'
                }
            }
        }
    }
}
