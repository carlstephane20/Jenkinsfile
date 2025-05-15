pipeline {
  agent any

  tools {
    nodejs 'Node 16'
  }

  environment {
    // Chemin de déploiement fictif sur un serveur Windows
    DEPLOY_PATH = 'C:\\deploy\\mon-app'
  }

  stages {
    stage('Checkout') {
      steps {
        git(
          url: 'https://github.com/mon-org/mon-app-node.git',
          credentialsId: 'github-nodejs-creds',
          branch: 'main'
        )
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test'
      }
      post {
        always {
          junit '**/test-results/*.xml'
        }
      }
    }

    stage('Build') {
      steps {
        sh 'npm run build'
      }
    }

    stage('Archive Artifacts') {
      steps {
        archiveArtifacts artifacts: 'dist/**', fingerprint: true
      }
    }

    stage('Deploy on Windows Server') {
      steps {
        bat """
          xcopy /E /Y dist\\* "%DEPLOY_PATH%\\"
        """
      }
    }
  }

  post {
    success {
      emailext to: 'team@domaine.com',
               subject: "✅ Build #${env.BUILD_NUMBER} réussi",
               body: "Voir la console : ${env.BUILD_URL}"
    }
    failure {
      emailext to: 'team@domaine.com',
               subject: "❌ Build #${env.BUILD_NUMBER} échoué",
               body: "Voir la console : ${env.BUILD_URL}"
    }
  }
}
