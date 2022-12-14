node {
        stage('git checkout') {
                git 'https://github.com/chkvijay/python-django-project.git'
               }
            
       stage('copy files to ansible server'){
            sshagent(['Ansible-server']) {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230'
             sh 'scp -r /var/lib/jenkins/workspace/pipeline-build/* ubuntu@172.31.45.230:/home/ubuntu/python'
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 rm -rf /home/ubuntu/python/*.yml'
	     sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 rm -rf /home/ubuntu/python/Jenkins'
             sh 'scp -r /var/lib/jenkins/workspace/pipeline-build/ansible_playbook.yml ubuntu@172.31.45.230:/home/ubuntu'
                }  
       }
            
        stage('build docker image'){
            sshagent(['Ansible-server']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 docker build -t $JOB_NAME:v1.$BUILD_ID python/.'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 rm -rf /home/ubuntu/python/*'
            }
        }
        
        stage('docker image tagging'){
            sshagent(['Ansible-server']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 docker tag $JOB_NAME:v1.$BUILD_ID chkvijay/$JOB_NAME:v1.$BUILD_ID'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 docker tag $JOB_NAME:v1.$BUILD_ID chkvijay/$JOB_NAME:latest'
           }
           }
        
        stage('docker image pushing to docker hub'){
            sshagent(['Ansible-server']) {
            withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub_passwd')]) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 docker login -u chkvijay -p ${dockerhub_passwd}"
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 docker push chkvijay/$JOB_NAME:v1.$BUILD_ID'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 docker push chkvijay/$JOB_NAME:latest'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 docker rmi $JOB_NAME:v1.$BUILD_ID chkvijay/$JOB_NAME:v1.$BUILD_ID chkvijay/$JOB_NAME:latest python:3'
                    }     
                  }
                  }
        
        stage('copy files from jenkins to minikube server'){
            sshagent(['minikube-server']) {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.41.47'
             sh 'scp -r /var/lib/jenkins/workspace/pipeline-build/*.yml ubuntu@172.31.41.47:/home/ubuntu'
                }   
		}

         stage('Run the Ansible-playbook'){
            sshagent(['Ansible-server']) {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.45.230 ansible-playbook ansible_playbook.yml'
             }
	     }
}
    
