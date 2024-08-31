pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                npm ci  #npm ci is used instead npm install as git won't include node_modules folder
                npm run build
                ls -la
                ls
                '''
            }
        }
    }
}