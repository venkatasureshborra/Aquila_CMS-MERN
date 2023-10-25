pipeline {
    agent any
       environment {
        BASTION_HOST = '3.110.225.14'
        WEB_SERVER_HOST = '10.0.12.66'
    }
    triggers {
        pollSCM('* * * * *') // Poll SCM every minute
    }
    options {
  buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '3')
    }

    stages {
        stage('CodeCheckout') {
            steps {
                git branch: "master", credentialsId: '12551f88-fa2c-44b5-aaef-7e301feb2294', url: 'https://github.com/venkatasureshborra/Aquila_CMS-MERN.git'
            }
        }
        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DockerHub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        def dockerLoginCmd = "docker login -u $USERNAME -p $PASSWORD"
                        def loginStatus = sh(script: dockerLoginCmd, returnStatus: true)
                        if (loginStatus != 0) {
                            error 'Docker login failed'
                        }
                    }
                }
            }
        }
        stage ('Docker_image build & push'){
            steps {
                sh " docker build -t venkat14489/aquilacms_mern:latest . "
                sh "docker push venkat14489/aquilacms_mern:latest "
            }
        }
         stage('Deploy to Web Server') {
            steps {
              script {
                    // Load the Bastion host's private key
                    withCredentials([file(credentialsId: 'bastion_host', variable: 'BASTION_PEM_FILE_PATH')]) {
                        // SSH to Bastion host
                        sshagent(['bastion_host']) {
                            // SSH to Web server from Bastion host
                            def sshCommand = """
                                ssh -A -o StrictHostKeyChecking=no -i ${BASTION_PEM_FILE_PATH}  ec2-user@${BASTION_HOST} \\
                                ssh -o StrictHostKeyChecking=no -i /home/ec2-user/Aquila_MERN-key.pem -tt  ubuntu@${WEB_SERVER_HOST} \\ <<EOF
                                docker ps -aq | xargs -r docker rm -f
                                docker run -d --name aquali -p 3010:3010 venkat14489/aquilacms_mern:latest
                                exit
                                EOF
                                 """
                            sh(sshCommand)
                        }
                    }
                }
            }
        }
        
    }
}
