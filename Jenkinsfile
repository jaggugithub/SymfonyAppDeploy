pipeline {
    agent any
    stages {
        stage('Code Deployment') {
            steps {
                sshagent(['k8smaster']) {
                    // This is to Copy a file From Jenkins Server to k8s master node
                    sh "scp -o StrictHostKeyChecking=no symfony-deploy.yaml ubuntu@<target server public IP>:/home/ubuntu/deployment"
                    sh "scp -o StrictHostKeyChecking=no symfony-deploy-service.yaml ubuntu@<target server public IP>:/home/ubuntu/deployment"
                }
                
            }
        }
        stage('Application Deployment') {
            steps {
                sshagent(['k8smaster']) {
                    // This is to create deployment and service objects in k8s cluster on ubuntu machine(AWS)(Name:k8smaster) from jenkins pipeline
                    sh """ssh -tt -o StrictHostKeyChecking=no ubuntu@<target server public IP> << EOF
                        cd /home/ubuntu/deployment
                        sudo kubectl create -f symfony-deploy.yaml
                        sudo kubectl create -f symfony-deploy-service.yaml 
                        exit
                        EOF"""
                }
                
            }
        }
    } 
}