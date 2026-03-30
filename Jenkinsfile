pipeline {
    agent any

    environment {
        APP_DIR = "/var/lib/jenkins/workspace/Flask-chat-app"
        VENV_DIR = "${APP_DIR}/venv"
        APP_PORT = "5000"
        LOG_FILE = "${APP_DIR}/flask.log"
        CRED_ID = "github-ssh-key" // Your Jenkins SSH credential ID
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'git@github.com:nirmal-debug995/myfullstackpython.git',
                        credentialsId: "${CRED_ID}"
                    ]]
                ])
            }
        }

        stage('Setup Environment') {
            steps {
                sh """
                    cd ${APP_DIR}
                    rm -rf ${VENV_DIR}
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                """
            }
        }

        stage('Stop Old App') {
            steps {
                sh """
                    OLD_PID=\$(lsof -t -i:${APP_PORT})
                    if [ ! -z "\$OLD_PID" ]; then
                        kill -9 \$OLD_PID
                        echo "Stopped old app with PID \$OLD_PID"
                    else
                        echo "No old app running on port ${APP_PORT}"
                    fi
                """
            }
        }

        stage('Start App') {
            steps {
                sh """
                    cd ${APP_DIR}
                    . ${VENV_DIR}/bin/activate
                    nohup python app.py --host=0.0.0.0 --port=${APP_PORT} > ${LOG_FILE} 2>&1 &
                    echo "Flask app started on port ${APP_PORT}, logs in ${LOG_FILE}"
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}
