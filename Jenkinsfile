pipeline {
    agent any
    environment {
        LIBLAB_TOKEN = credentials('LIBLAB_TOKEN')
        REPO_HOST_PLATFORM = credentials('REPO_HOST_PLATFORM')
        LIBLAB_GITHUB_TOKEN = credentials('LIBLAB_GITHUB_TOKEN')
        LIBLAB_BITBUCKET_TOKEN = credentials('LIBLAB_BITBUCKET_TOKEN')
        LIBLAB_GITLAB_TOKEN = credentials('LIBLAB_GITLAB_TOKEN')
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
                    if (!env.LIBLAB_TOKEN) {
                        error("Error: LIBLAB_TOKEN is not defined")
                    }
                    if (!env.REPO_HOST_PLATFORM) {
                        error("Error: REPO_HOST_PLATFORM is not defined")
                    }
                    
                    if(env.REPO_HOST_PLATFORM == 'github' && !env.LIBLAB_GITHUB_TOKEN) {
                        error("Error: LIBLAB_GITHUB_TOKEN is not defined")
                    }
                    if(env.REPO_HOST_PLATFORM == 'bitbucket' && !env.LIBLAB_BITBUCKET_TOKEN) {
                        error("Error: LIBLAB_BITBUCKET_TOKEN is not defined")
                    }
                    if(env.REPO_HOST_PLATFORM == 'gitlab' && !env.LIBLAB_GITLAB_TOKEN) {
                        error("Error: LIBLAB_GITLAB_TOKEN is not defined")
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
