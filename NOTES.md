\# DevOps CI/CD Pipeline Project Notes



\## 1. Project Overview



This project implements a CI/CD pipeline using Jenkins, Docker, Flask, pytest, Prometheus, Pushgateway, and Grafana.



\## 2. Application



\- Framework: Flask

\- Health endpoint: `/health`

\- Addition endpoint: `/add?a=5\&b=3`

\- Automated tests: pytest

\- Code linting: flake8



\## 3. Environment Setup



The project runs using Docker Compose.



Services and local URLs:



\- Jenkins: http://localhost:8080

\- Grafana: http://localhost:3000

\- Prometheus: http://localhost:9090

\- Pushgateway: http://localhost:9091



Start the services from the project root:



```powershell

docker compose up -d --build

```



Check service status:



```powershell

docker compose ps

```



\## 4. Jenkins CI Pipeline



Jenkins job: `DevOps-CI-Pipeline`



Pipeline stages include:



1\. Checkout source code

2\. Lint Python code using flake8

3\. Run automated tests using pytest

4\. Build the Docker image

5\. Run a smoke test against the application



The pipeline also publishes JUnit test results and cleans up the test container.



\## 5. Monitoring and Metrics



Pipeline metrics are published to Pushgateway and collected by Prometheus.



Metrics:



\- `pipeline\_build\_status`: 1 for success, 0 for failure

\- `pipeline\_build\_duration\_seconds`: build duration in seconds

\- `pipeline\_tests\_passed`: number of passed tests

\- `pipeline\_tests\_failed`: number of failed tests



Pushgateway job name: `interview\_pipeline`



\## 6. Grafana Dashboard



Dashboard name: `CI pipeline health`



Panels:



1\. Last build status

2\. Build duration over time

3\. Tests passed and failed



Prometheus data source URL inside the Docker Compose network:



`http://prometheus:9090`



Dashboard JSON file:



`monitoring/grafana/dashboards/ci-pipeline.json`



\## 7. Verification



The Jenkins pipeline was verified with a successful build. The recorded test results were 5 passed and 0 failed.



The custom pipeline metrics were successfully published to Pushgateway, and the Grafana dashboard was confirmed to display data.



\## 8. GitHub Repository



Repository: https://github.com/Madhu655379/devopsjr



Branch: `main`



