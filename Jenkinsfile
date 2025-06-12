pipeline {
    agent {
        docker {
            image 'yourdockerhubuser/jenkins-agent-with-ansible'
            args '-u root'
        }
    }

    environment {
        IMAGE_NAME = "shay-flask-app"
        IMAGE_TAG = "latest"
        CONTAINER_NAME = "flask-container"
    }

    stages {
        stage('Clone Repo') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Run Container') {
            steps {
                sh "docker run -d --rm --name ${CONTAINER_NAME} -p 5001:5001 ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Test') {
            steps {
                echo "Waiting for app to start..."
                sh "sleep 3"
                echo "Running test on http://localhost:5001 ..."
                sh 'curl --fail http://localhost:5001 || (echo "❌ Test failed!" && exit 1)'
            }
        }

        stage('Cleanup') {
            steps {
                sh "docker stop ${CONTAINER_NAME} || true"
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh '''
                ansible-playbook -i ansible/hosts.ini ansible/deploy.yml \
                    --extra-vars "image_name=${IMAGE_NAME} image_tag=${IMAGE_TAG}"
                '''
            }
        }
    }

    post {
        success {
            echo 'App test passed. Build OK.'
        }
        failure {
            echo 'App test failed. Build FAILED.'
        }
        always {
            echo 'Pipeline finished.'
        }
    }
}
