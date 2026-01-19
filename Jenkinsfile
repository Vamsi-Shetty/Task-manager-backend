pipeline {
    agent any

    environment {
        NODE_VERSION = '18'
        npm_config_cache = '${WORKSPACE}/.npm'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    env.GIT_BRANCH_NAME = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()
                }
            }
        }

        stage('Setup') {
            steps {
                script {
                    echo "Setting up Node.js environment..."
                    sh '''
                        node --version
                        npm --version
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing project dependencies..."
                    sh 'npm ci --prefer-offline --no-audit'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building TypeScript project..."
                    sh 'npm run build'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Running tests..."
                    sh 'npm run test -- --run --reporter=verbose'
                }
            }
            post {
                always {
                    junit testResults: 'test-results.xml', allowEmptyResults: true
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    echo "Verifying build output..."
                    sh '''
                        if [ -f dist/index.js ]; then
                            echo "✓ Build artifact exists: dist/index.js"
                        else
                            echo "✗ Build artifact missing!"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                script {
                    echo "Archiving build artifacts..."
                    sh 'tar -czf dist.tar.gz dist/'
                    archiveArtifacts artifacts: 'dist.tar.gz', allowEmptyArchive: false
                }
            }
        }
    }

    post {
        success {
            script {
                echo "✓ Build ${BUILD_NUMBER} succeeded on ${GIT_BRANCH_NAME} (${GIT_COMMIT_SHORT})"
            }
        }
        failure {
            script {
                echo "✗ Build ${BUILD_NUMBER} failed on ${GIT_BRANCH_NAME} (${GIT_COMMIT_SHORT})"
            }
        }
        always {
            cleanWs()
        }
    }
}
