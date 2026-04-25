pipeline {
    agent any

    environment {
        GIT_REPO_URL = 'https://github.com/volarevin/cicd.git'
        GIT_CREDENTIALS_ID = 'github-pat'
        GIT_BRANCH = 'main'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm: [
                    $class: 'GitSCM',
                    branches: [[name: "*/${env.GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${env.GIT_REPO_URL}",
                        credentialsId: "${env.GIT_CREDENTIALS_ID}"
                    ]]
                ]
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                echo "Setting up Python environment..."

                python3 -m venv venv
                . venv/bin/activate

                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Run Selenium Test') {
            steps {
                sh '''
                echo "Running Selenium tests..."

                . venv/bin/activate
                python test.py
                '''
            }
        }

        stage('Deploy to Apache') {
            steps {
                sh '''
                echo "Deploying FULL PHP project to Apache..."

                # Sync all files (NEW + UPDATED + DELETED)
                sudo rsync -av -o --delete ./ /var/www/html/

                # Fix ownership
                sudo chown -R www-data:www-data /var/www/html/
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD SUCCESS ✔ Deployment completed"
        }
        failure {
            echo "CI/CD FAILED ❌ Check logs"
        }
        always {
            cleanWs()
        }
    }
}
