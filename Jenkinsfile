pipeline {
    agent any

    stages {
        stage('Prepare nginx VM') {
            steps {
                sh '''
                #!/bin/bash
                echo "connecting to remote machine terraform-execution"
                ssh -tty jenkins@10.190.0.3 << EOF

                echo "installing terraform if not present"

                if test -f "/usr/bin/terraform"; then
                echo "terraform already exists, not installing"
                else
                echo "installing terraform"
                wget https://releases.hashicorp.com/terraform/1.0.3/terraform_1.0.3_linux_amd64.zip
                unzip terraform_1.0.3_linux_amd64.zip
                sudo mv terraform /usr/bin/terraform
                terraform version
                fi

                if [ ! -d "nginx-terraform" ] ; then
                git clone https://github.com/RiflerRick/nginx-terraform
                else
                cd nginx-terraform
                git pull origin main
                fi
                cp /home/jenkins/svc-account-key.json svc-account-key.json
                terraform init
                terraform apply -auto-approve -no-color
                exit 0
                EOF
                '''
            }
        }
        stage('Install nginx in VM') {
            steps {
                sh '''
                #!/bin/bash
                echo "connecting to remote machine terraform-execution"
                ssh -tty jenkins@10.190.0.3 << EOF
                if [ ! -d "nginx-ansible" ] ; then
                git clone https://github.com/RiflerRick/nginx-ansible
                else
                cd nginx-ansible
                git pull origin main
                fi
                pip3 install -r requirements.txt
                #intIP=$(gcloud compute instances describe nginx-server --zone=asia-south2-a --format="yaml(networkInterfaces[0].networkIP)" | grep networkIP | cut -d ':' -f 2 | sed "s/ //g")
                #python3 generate_hosts_file.py $intIP
                ansible-playbook -i hosts webservers.yml
                exit 0
                EOF
                '''
            }
        }
        stage('Test nginx') {
            steps {
                sh '''
                #!/bin/bash
                echo "testing nginx server using public ip address"
                extIP=$(gcloud compute instances describe nginx-server --format="yaml(networkInterfaces[0].accessConfigs[0].natIP)" | grep natIP | cut -d ':' -f 2 | sed "s/ //g")
                curl $extIP
                status_code=$(curl --connect-timeout 5 -s -o /dev/null -w "%{http_code}" $extIP)
                if [[ $status_code != 200 ]] ; then
                echo "nginx could not be reached on $extIP"
                exit 1
                fi
                exit 0
                '''
            }
        }
    }
}