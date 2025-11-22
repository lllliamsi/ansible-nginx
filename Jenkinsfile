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

        stage('Start Dummy Target') {
            steps {
                sh '''
                docker run -d --name ansible-test \
                    -p 2222:22 \
                    dummy-ansible
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

    post {
        always {
            sh "docker rm -f ansible-test || true"
            echo "Pipeline completed."
        }
    }
}
