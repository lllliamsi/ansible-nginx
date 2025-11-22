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
            echo "Pipeline completed."
        }
    }
}
