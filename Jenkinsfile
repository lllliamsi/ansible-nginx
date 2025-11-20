pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Lint Playbook') {
            steps {
                sh '''
                    echo "Running ansible-lint..."
                    ansible-lint playbooks/install-nginx.yaml
                '''
            }
        }

        stage('Syntax Check') {
            steps {
                sh '''
                    echo "Running syntax check..."
                    ansible-playbook playbooks/install-nginx.yaml --syntax-check -i inventory/dev
                '''
            }
        }

        stage('Prepare Dummy Container') {
            steps {
                sh '''
                    echo "Creating dummy container for simulation..."
                    docker run -d --name ansible-test --rm ubuntu:22.04 tail -f /dev/null
        
                    echo "Installing dependencies and setting up SSH..."
                    docker exec ansible-test bash -c '
                        apt-get update &&
                        apt-get install -y openssh-server python3 sudo &&
                        mkdir -p /var/run/sshd &&
                        echo "root:root" | chpasswd &&
                        sed -i "s/^PermitRootLogin.*/PermitRootLogin yes/" /etc/ssh/sshd_config &&
                        sed -i "s/^#PasswordAuthentication.*/PasswordAuthentication yes/" /etc/ssh/sshd_config &&
                        sed -i '/^Include .*sshd_config\.d/d' /etc/ssh/sshd_config || true
                        service ssh restart
                    '
        
                    # beri waktu SSH ready
                    sleep 3
        
                    # ambil IP container
                    CONTAINER_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ansible-test)
        
                    # buat inventory dummy
                    mkdir -p inventory
                    echo "[dummy]" > inventory/dummy
                    echo "$CONTAINER_IP ansible_user=root ansible_password=root ansible_ssh_common_args='-o StrictHostKeyChecking=no'" >> inventory/dummy
                '''
            }
        }


        stage('Dry Run (Check Mode)') {
            steps {
                sh '''
                    echo "Running playbook in check mode against dummy container..."
                    # ambil IP dari inventory dummy
                    CONTAINER_IP=$(awk 'NR==2 {print $1}' inventory/dummy)
        
                    # hapus host lama dari known_hosts
                    ssh-keygen -f "/var/jenkins_home/.ssh/known_hosts" -R "$CONTAINER_IP" || true
        
                    # jalankan ansible playbook
                    ansible-playbook playbooks/install-nginx.yaml --check -i inventory/dummy -vvv
                '''
            }
        }
    }

    post {
        always {
            sh "docker rm -f ansible-test || true"
            echo "Pipeline completed."
        }
    }
}
