pipeline {
    agent any

    environment {
        VENV_DIR = "${WORKSPACE}/venv"
        APP_NAME = "app.py"
    }

    stages {
        stage('Clone Code') {
            steps {
                sshagent(['github-ssh-key']) {
                    sh '''
                    if [ -d "Flask-Chat-App" ]; then
                        cd Flask-Chat-App
                        git pull origin main
                    else
                        git clone -b main git@github.com:Deshan555/Flask-Chat-App.git
                    fi
                    '''
                }
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                cd Flask-Chat-App

                rm -rf $VENV_DIR
                python3 -m venv $VENV_DIR

                source $VENV_DIR/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Stop Old App') {
            steps {
                sh '''
                pkill -f $APP_NAME || echo "No running app"
                '''
            }
        }

        stage('Start App') {
            steps {
                sh '''
                cd Flask-Chat-App
                source $VENV_DIR/bin/activate

                nohup python $APP_NAME > app.log 2>&1 &
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}
