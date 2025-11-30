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

        stage('Prepare Dummy Container') {
            steps {
                sh '''        
                    echo "Starting dummy container..."
                    docker run -d --name ansible-dummy \
                        --privileged \
                        --network jenkins-net \
                        -p 2222:22 \
                        ansible-dummy:latest
        
                    echo "Waiting for SSH..."
                    sleep 8
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

        stage('Check Mode') {
            steps {
                sh '''
                    ansible-playbook playbooks/install-nginx.yaml \
                        -i inventory/dummy \
                        --check -vvv
                '''
            }
        }
    }

    post {
        always {
            sh '''
                echo "Cleaning dummy container..."
                docker rm -f ansible-dummy || true
            '''
            echo "Pipeline completed."
        }
    }
}
