pipeline {
    agent any
    environment {
        LIBLAB_TOKEN = credentials('LIBLAB_TOKEN')
        LIBLAB_BITBUCKET_TOKEN = credentials('LIBLAB_BITBUCKET_TOKEN')
        REPO_HOST_PLATFORM = credentials('REPO_HOST_PLATFORM')
    }
    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }
        stage('Pre-check for Environment Variables') {
            steps {
                script {
                    if (!env.LIBLAB_TOKEN || !env.LIBLAB_BITBUCKET_TOKEN) {
                        error("Error: LIBLAB_TOKEN or LIBLAB_BITBUCKET_TOKEN is not defined")
                    }
                    if (!env.REPO_HOST_PLATFORM) {
                        error("Error: REPO_HOST_PLATFORM is not defined")
                    }
                    echo "Environment variables are defined."
                }
            }
        }
        stage('Install Node.js and npm') {
            steps {
                script {
                    sh '''
                    curl -sL https://deb.nodesource.com/setup_20.x | bash -
                    apt-get install -y nodejs
                    node -v
                    npm -v
                    '''
                }
            }
        }
        stage('Check for File Changes') {
            steps {
                script {
                    def changes = sh(script: 'git diff --quiet HEAD~1 -- liblab.config.json spec/ hooks/ customPlanModifiers/ || echo "CHANGES"', returnStdout: true).trim()
                    if (changes != "CHANGES") {
                        echo "No changes detected in relevant files. Skipping pipeline."
                        currentBuild.result = 'SUCCESS'
                        error("Stopping pipeline as no relevant files were changed")
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    sh 'npm install -g liblab'
                }
            }
        }
        stage('Build SDKs and Create Pull Request') {
            steps {
                script {
                    sh('liblab build --yes --pr -p $REPO_HOST_PLATFORM')
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
