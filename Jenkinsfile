pipeline {
    agent any

    stages {
        stage('Build and Push Application') {
            steps {
                script {
                    docker.image('docker:27.4.0').inside('--user root -v /var/run/docker.sock:/var/run/docker.sock') {
                        sh """   
                            docker login -u your_login -p yuor_token https://index.docker.io/v1/  
                            docker build -f /var/lib/jenkins/workspace/deploy-app/Dockerfile -t aamitv/app-ci-cd .
                            docker push aamitv/app-ci-cd
                        """
                    }
                }
            }
        }
    
       stage('Run instance on yandex-cloud') {
         
         steps {
            sh 'terraform init'
            sh 'terraform apply -auto-approve'
         }
    }

       stage('Generate Ansible Inventory') {
         steps {
            script {
                def prodNodeIp = sh(script: 'terraform output -raw external_ip_address_prod-stand', returnStdout: true).trim() // Get production node IP

                writeFile file: 'hosts', text: """
                       [prod-stand]
                       ${prodNodeIp} ansible_user=jenkins ansible_ssh_common_args='-o StrictHostKeyChecking=no -o ConnectionAttempts=10 -o ConnectTimeout=15'
                    """ // Create Ansible inventory file
                }
            }
        }
       
 
       stage('Deploy app') {
        steps {
            script {
                sh 'ansible-playbook -i hosts prod-stand.yaml'
                }
            }
        }
    }
}
