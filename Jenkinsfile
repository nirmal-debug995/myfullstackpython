pipeline {
    agent any

    environment {
        VENV_DIR = "${WORKSPACE}/venv"
        APP_PORT = 5000
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'git@github.com:nirmal-debug995/myfullstackpython.git',
                        credentialsId: 'github-ssh-key'
                    ]]
                ])
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                #!/bin/bash
                set -e
                set -x

                # Remove old venv
                rm -rf "${VENV_DIR}"

                # Create virtualenv
                python3 -m venv "${VENV_DIR}"

                # Activate virtualenv
                source "${VENV_DIR}/bin/activate"

                # Upgrade pip and install requirements
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Stop Old App') {
            steps {
                sh '''
                #!/bin/bash
                set +e  # allow command to fail if nothing is running

                OLD_PID=$(lsof -t -i:${APP_PORT})
                if [ -n "$OLD_PID" ]; then
                    echo "Stopping old app (PID $OLD_PID)..."
                    kill -9 $OLD_PID
                else
                    echo "No existing app running on port ${APP_PORT}"
                fi
                '''
            }
        }

        stage('Start App') {
            steps {
                sh '''
                #!/bin/bash
                set -e

                # Activate venv
                source "${VENV_DIR}/bin/activate"

                # Start Flask app with Gunicorn + Eventlet
                gunicorn -w 1 -k eventlet -b 0.0.0.0:${APP_PORT} app:app &
                echo "App started on port ${APP_PORT}"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment succeeded"
        }
        failure {
            echo "❌ Deployment failed. Check logs for details."
        }
    }
}
