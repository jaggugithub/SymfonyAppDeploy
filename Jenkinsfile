pipeline {
    agent any
    stages {
        stage('Get Master Public IP') {
            steps {
                dir('/var/lib/jenkins/workspace/k8sinfra') {
                    sh '''
                    terraform output master_public_ip | sed -n '2p' | sed -e 's/^[ \t]*//' -e 's/,//'  -e 's/^"//' -e 's/"//' >> file.txt
                    '''
                    script {
                        // trim removes leading and trailing whitespace from the string
                        masterip = readFile('file.txt').trim()
                    }
                echo "${masterip}" // prints 'IP'
                }
            }
        }
        stage('Code Deployment') {
            steps {
                sshagent(['aws_ssh_key']) {
                    // This is to Copy a file From Jenkins Server to k8s master node
                    sh """ssh -tt -o StrictHostKeyChecking=no ubuntu@${masterip} << EOF
                    mkdir deployment
                    EOF"""
                    sh "scp -o StrictHostKeyChecking=no symfony-deploy.yaml ubuntu@${masterip}:/home/ubuntu/deployment"
                    sh "scp -o StrictHostKeyChecking=no symfony-deploy-service.yaml ubuntu@${masterip}:/home/ubuntu/deployment"
                }
                
            }
        }
        stage('Application Deployment') {
            steps {
                sshagent(['aws_ssh_key']) {
                    // This is to create deployment and service objects in k8s cluster on ubuntu machine(AWS)(Name:k8smaster) from jenkins pipeline
                    sh """ssh -tt -o StrictHostKeyChecking=no ubuntu@${masterip} << EOF
                        cd /home/ubuntu/deployment
                        sudo docker pull jaggu199/symfony:2
                        sudo kubectl create -f symfony-deploy.yaml
                        sudo kubectl create -f symfony-deploy-service.yaml 
                        exit
                        EOF"""
                }
                
            }
        }
    } 
}