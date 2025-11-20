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
                    docker exec ansible-test bash -c "
                      apt-get update && apt-get install -y openssh-server python3 sudo && \
                      mkdir -p /root/.ssh && \
                      ssh-keygen -t ed25519 -f /root/.ssh/id_ed25519 -N '' && \
                      cat /root/.ssh/id_ed25519.pub >> /root/.ssh/authorized_keys && \
                      chmod 600 /root/.ssh/authorized_keys
                    "
                    docker exec ansible-test service ssh restart

                    # beri waktu SSH ready
                    sleep 3
        
                    # ambil IP container
                    CONTAINER_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ansible-test)
        
                    # buat inventory dummy
                    mkdir -p inventory
                    echo "[dummy]" > inventory/dummy
                    echo "$CONTAINER_IP ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_ed25519 ansible_ssh_common_args='-o StrictHostKeyChecking=no'" >> inventory/dummy
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
                    ansible-playbook playbooks/install-nginx.yaml --check -i inventory/dummy
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
