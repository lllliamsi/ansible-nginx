pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y ansible ansible-lint docker.io
                '''
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

                    # copy SSH key or use docker exec to install sshd
                    docker exec ansible-test apt-get update
                    docker exec ansible-test apt-get install -y openssh-server python3

                    # enable ssh
                    docker exec ansible-test service ssh start

                    # get container IP
                    CONTAINER_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ansible-test)
                    echo "[dummy]\n$CONTAINER_IP ansible_user=root ansible_password=root ansible_ssh_common_args='-o StrictHostKeyChecking=no'" > inventory/dummy
                '''
            }
        }

        stage('Dry Run (Check Mode)') {
            steps {
                sh '''
                    echo "Running playbook in check mode against dummy container..."
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
