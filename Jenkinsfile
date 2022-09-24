pipeline {
    agent any
    stages {
        stage('Get Buildnum & Master Public IP') {
            steps {
                    script {
                        // trim removes leading and trailing whitespace from the string
                        masterip = readFile('/var/lib/jenkins/workspace/k8sinfra/ip.txt').trim()
                        appbuild = readFile('/var/lib/jenkins/workspace/symfony-app-build/buildnum.txt').trim()
                    }
                echo "${masterip}" // prints 'k8s Master IP'
                echo "${appbuild}" // prints 'App Build Number'
            }
        }
        stage('Get Build Number in deploy file') {
            steps {
                sh """
                    sed -i "s/appbuildnum/$appbuild/g" symfony-deploy.yaml
                    """
            }
        }
        stage('Code Deployment') {
            steps {
                sshagent(['aws_ssh_key']) {
                    // This command is to create a deployment directory on k8s master node
                    sh "ssh -tt -o StrictHostKeyChecking=no ubuntu@${masterip} mkdir deployment"
                    // The below commands are used to Copy files From Jenkins Server to k8s master node
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
                        sudo docker pull jaggu199/symfony:${appbuild}
                        sudo kubectl create -f symfony-deploy.yaml
                        sudo kubectl create -f symfony-deploy-service.yaml
                        exit
                        EOF"""
                }
                
            }
        }
    }
// If you want to clean the workspace for this pipeline job please uncomment the post stages section(**BUT NOT THIS LINE**).
    // post {
	// 	always {
	// 	    cleanWs()    // This is to clean the workspace for this job
	//     }
 	// } 
}