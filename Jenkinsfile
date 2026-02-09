pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
    - name: terraform
      image: hashicorp/terraform:1.6
      command: ['cat']
      tty: true
      volumeMounts:
        - name: ssh-key-volume
          mountPath: /home/jenkins/.ssh
          readOnly: true
    - name: ansible
      image: alpine/ansible:latest
      command: ['cat']
      tty: true
      volumeMounts:
        - name: ssh-key-volume
          mountPath: /home/jenkins/.ssh
          readOnly: true
  volumes:
    - name: ssh-key-volume
      secret:
        secretName: jenkins-ssh-key
"""
        }
    }

    stages {
        stage('Checkout Repo') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: 'https://github.com/sanchitk2002/jenkins-eks.git']]
                ])
            }
        }

        stage('Terraform Apply') {
            steps {
                container('terraform') {
                    sh 'terraform -chdir=terraform init'
                    sh 'terraform -chdir=terraform apply --auto-approve'
                    script {
                        env.WEB_IP = sh(
                            script: 'terraform -chdir=terraform output -raw web_instance_ip',
                            returnStdout: true
                        ).trim()
                        echo "Web server IP: ${env.WEB_IP}"
                    }
                }
            }
        }
        
        

        stage('Run Ansible') {
            steps {
                container('ansible') {
                    sh """
                    mkdir -p ansible
                    echo "[web]" > ansible/inventory
                    echo "${WEB_IP}" >> ansible/inventory
                    cp /home/jenkins/.ssh/jenkins.pem /tmp/jenkins.pem
                    chmod 600 /tmp/jenkins.pem
                    export ANSIBLE_HOST_KEY_CHECKING=False
                     # Wait for SSH to be ready
                       until nc -z -w5 ${WEB_IP} 22; do
                       echo "Waiting for SSH..."
                       sleep 5
                        done

                    ansible-playbook -i ansible/inventory ansible/nginx.yaml --private-key=/tmp/jenkins.pem -u ubuntu
                    """
                }
            }
        }
    }
}
