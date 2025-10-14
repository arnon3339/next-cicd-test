pipeline {
  agent { label 'docker-agent' }

  environment {
    IMAGE     = "nextjs:${env.GIT_COMMIT.take(7)}"
    CONTAINER = 'nextjs'
    PORT      = '3000'
  }

  triggers {
    githubPush()
  }

  stages {
    stage('Checkout') {
      when { branch 'main' }
      steps { checkout scm }
    }

    stage('Build image') {
      when { branch 'main' }
      steps {
        withCredentials([file(credentialsId: 'github-test-dockerfile', variable: 'DOCKER_FILE')]) {
          sh '''#!/usr/bin/env bash
            set -euo pipefail
            docker build -t "${IMAGE}" -f "${DOCKER_FILE}" .
            docker tag "${IMAGE}" nextjs:latest
          '''
        }
      }
    }

    stage('Deploy') {
      when { branch 'main' }
      steps {
        sh '''#!/usr/bin/env bash
          set -euo pipefail
          docker rm -f "${CONTAINER}" 2>/dev/null || true
          docker rmi -f nextjs:latest 2>/dev/null || true
          docker run -d --name "${CONTAINER}" \
            -p "${PORT}:3000" \
            --restart unless-stopped \
            "${IMAGE}"
        '''
      }
    }
  }

  post {
    success { echo "✅ Deployed ${CONTAINER} from ${env.GIT_COMMIT.take(7)}" }
    failure { echo "❌ Deploy failed — check the console log" }
  }
}
