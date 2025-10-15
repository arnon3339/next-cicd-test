pipeline {
  agent { label 'docker-agent' }

  environment {
    IMAGE         = "nextjs:${env.GIT_COMMIT.take(7)}"
    CONTAINER     = "nextjs"
    PORT          = "3000"
    APP_URL       = "https://aoc.dataxo.info"
    GIT_REPO_URL  = "https://github.com/arnon3339/next-cicd-test"
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
        // error('💥 Simulated failure for notification testing')
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
    success {
      withCredentials([string(credentialsId: 'aoc-discord-webhook', variable: 'DISCORD_URL')]) {
        sh '''
          SHORT_SHA=$(printf "%s" "$GIT_COMMIT" | cut -c1-7)
          curl -sSf \
          -H "Content-Type: application/json" \
          --data @- \
          "$DISCORD_URL" <<EOF
{
  "username": "Jenkins CI/CD",
  "avatar_url": "${APP_URL}/mule3.png",
  "embeds": [{
    "title": "✅ Deployment Successful!",
    "description": "**Application has been successfully deployed.**\\n\\n[Visit the app](${APP_URL})",
    "color": 5814783,
    "fields": [
      { "name": "Commit", "value": "[${SHORT_SHA}](${GIT_REPO_URL}/tree/${GIT_COMMIT})", "inline": true }
    ],
    "image": { "url": "${APP_URL}/mule-logo.png" },
    "footer": { "text": "Deployed by Jenkins", "icon_url": "${APP_URL}/Jenkins_logo.svg.png" },
    "timestamp": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
  }]
}
EOF

          echo "✅ Deployed ${CONTAINER}"
        '''
      }
    }

    failure {
      withCredentials([string(credentialsId: 'aoc-discord-webhook', variable: 'DISCORD_URL')]) {
        sh '''
          SHORT_SHA=$(printf "%s" "$GIT_COMMIT" | cut -c1-7)
          curl -sSf \
          -H "Content-Type: application/json" \
          --data @- \
          "$DISCORD_URL" <<EOF
{
  "username": "Jenkins CI/CD",
  "avatar_url": "${APP_URL}/mule3.png",
  "embeds": [{
    "title": "❌ Deployment Failed!",
    "description": "**Application has been fail to be deployed.**\\n\\n",
    "color": 15158332,
    "fields": [
      { "name": "Commit", "value": "[${SHORT_SHA}](${GIT_REPO_URL}/tree/${GIT_COMMIT})", "inline": true }
    ],
    "image": { "url": "${APP_URL}/mule-fail.png },
    "timestamp": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
  }]
}
EOF

          echo "❌ Deploy failed — check the console log"
        '''
      }
    }
  }

}
