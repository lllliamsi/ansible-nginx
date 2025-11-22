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

        stage('Debug') {
            steps {
                sh '''
                    echo "PWD: $(pwd)"
                    echo "List root:"
                    ls -lah
                    echo "List playbooks:"
                    ls -lah playbooks || echo "Playbooks missing!"
                    echo "List inventory:"
                    ls -lah inventory || echo "Inventory missing!"
                '''
            }
        }

        stage('Run Ansible in Container') {
            steps {
                sh """
                    echo "WORKSPACE = $WORKSPACE"
                    docker run --rm \
                        -v "$WORKSPACE:/ansible" \
                        -w /ansible \
                        ubuntu:22.04 bash -c '
                            apt-get update &&
                            DEBIAN_FRONTEND=noninteractive apt-get install -y ansible &&
                            ansible-playbook playbooks/install-nginx.yaml --check -i inventory/local
                        '
                """
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
