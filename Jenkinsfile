
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

            script {
                def buildStatus =
                    currentBuild.currentResult == 'SUCCESS' ? 1 : 0
                def durationSeconds =
                    currentBuild.duration / 1000.0

                withEnv([
                    "PIPELINE_BUILD_STATUS=${buildStatus}",
                    "PIPELINE_BUILD_DURATION_SECONDS=${durationSeconds}"
                ]) {
                    sh '''
                        python3 - <<'PY'
import os
import glob
import xml.etree.ElementTree as ET

passed = 0
failed = 0

for filename in glob.glob("reports/junit.xml"):
    try:
        root = ET.parse(filename).getroot()
        cases = root.iter("testcase")

        for case in cases:
            if case.find("failure") is not None or case.find("error") is not None:
                failed += 1
            elif case.find("skipped") is None:
                passed += 1
    except (ET.ParseError, OSError):
        pass

with open("/tmp/pipeline-metrics.prom", "w") as metrics:
    metrics.write("# TYPE pipeline_build_status gauge\\n")
    metrics.write(f"pipeline_build_status {os.environ['PIPELINE_BUILD_STATUS']}\\n")
    metrics.write("# TYPE pipeline_build_duration_seconds gauge\\n")
    metrics.write(f"pipeline_build_duration_seconds {os.environ['PIPELINE_BUILD_DURATION_SECONDS']}\\n")
    metrics.write("# TYPE pipeline_tests_passed gauge\\n")
    metrics.write(f"pipeline_tests_passed {passed}\\n")
    metrics.write("# TYPE pipeline_tests_failed gauge\\n")
    metrics.write(f"pipeline_tests_failed {failed}\\n")
PY

                        echo "Publishing pipeline metrics to Pushgateway..."

                        curl --fail --silent --show-error \
                            --retry 3 \
                            --data-binary @/tmp/pipeline-metrics.prom \
                            http://pushgateway:9091/metrics/job/interview_pipeline

                        echo "Metrics published successfully."
                    '''
                }
            }

            sh '''
                docker rm -f "$CONTAINER_NAME" 2>/dev/null || true
            '''

            deleteDir()
        }
    }
}
