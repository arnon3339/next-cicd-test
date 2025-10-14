pipeline {
  agent { label 'docker-agent' }

  environment {
    IMAGE     = "nextjs:${env.GIT_COMMIT.take(7)}"
    CONTAINER = 'nextjs'
    PORT      = '3000'
  }

  stages {
    stage('Checkout') {
        when { branch 'main' }
        steps { checkout scm }
    }

    stage('Build image') {
      steps {
        sh '''#!/usr/bin/env bash
          set -euo pipefail
          docker build -t "${IMAGE}" .
          # also tag latest if you want
          docker tag "${IMAGE}" nextjs:latest
        '''
      }
    }

    stage('Deploy') {
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
