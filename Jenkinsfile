pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_NAME = 'interview-app'
        CONTAINER_NAME = "interview-app-${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: '/repo']]
                ])
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    pip install -r app/requirements.txt
                    flake8 app/
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . .venv/bin/activate
                    mkdir -p reports
                    pytest --junitxml=reports/junit.xml
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Smoke test') {
            steps {
                sh '''
                    docker run -d \
                        --name "$CONTAINER_NAME" \
                        --network devopsjr_default \
                        "$IMAGE_NAME:$BUILD_NUMBER"

                    sleep 3

                    docker exec "$CONTAINER_NAME" python -c \
                        "import urllib.request; r=urllib.request.urlopen('http://127.0.0.1:5000/health'); print(r.read().decode()); assert r.status == 200"
                '''
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'reports/junit.xml'

            sh '''
                docker rm -f "$CONTAINER_NAME" 2>/dev/null || true
            '''

            deleteDir()
        }
    }
}