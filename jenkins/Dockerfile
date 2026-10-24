
FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      docker-cli python3 python3-venv curl \
 && command -v docker \
 && docker --version \
 && rm -rf /var/lib/apt/lists/*

USER jenkins

RUN jenkins-plugin-cli --plugins \
    workflow-aggregator \
    git \
    junit \
    pipeline-stage-view \
    docker-workflow
