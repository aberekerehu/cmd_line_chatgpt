pipeline {
    agent any

    environment {
        PYTHON_EXEC = "${tool 'Python3'}"
        OPENAI_API_KEY = '<your OPENAI API Key>'
        ARTIFACTORY_SERVER_ID = 'your_artifactory_server_id'
        ARTIFACTORY_REPO = 'your_artifactory_repository'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/aberekerehu/cmd_line_chatgpt.git']])
            }
        }
        stage('Create .env File') {
            steps {
                script {
                    // Create .env file with OPENAI_API_KEY
                    sh 'echo "OPENAI_API_KEY=${OPENAI_API_KEY}" > .env'
                }
            }
        }
        stage('Activate Virtual Environment') {
            steps {
                script {
                    sh '${PYTHON_EXEC} -m venv venv'
                    // Activate the virtual environment created by the user
                    sh '. venv/bin/activate'
                }
            }
        }


        stage('Install Dependencies') {
            steps {
                sh "${PYTHON_EXEC} -m pip install -r requirements.txt"
                // Install flake8 for code analysis
                sh "${PYTHON_EXEC} -m pip install flake8"
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    // Run flake8 for static code analysis
                    def flake8Output = sh(
                        script: "${PYTHON_EXEC} -m flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics",
                        returnStatus: true
                    )

                    // Check if the count of errors and warnings is less than 50
                    if (flake8Output < 40) {
                        echo 'Code analysis passed. Number of errors and warnings is less than 50.'
                    } else {
                        error 'Code analysis failed. Number of errors and warnings is 50 or more.'
                    }
                }
            }
        }

        // stage('Test') {
        //     steps {
        //         script {
        //             // Run pytest for testing
        //             // sh "${PYTHON_EXEC} -m pytest tests/"
        //         }
        //     }
        // }

        stage('Deploy') {
            steps {
                script {
                    sh "${PYTHON_EXEC} gpt.py"
                }
            }
        }
        

        // stage('Upload to Artifactory') {
        //     when {
        //         expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
        //     }
        //     steps {
        //         script {
        //             // Upload artifacts to JFrog Artifactory only on success
        //             def server = Artifactory.server ARTIFACTORY_SERVER_ID
        //             def buildInfo = Artifactory.newBuildInfo()

        //             server.upload spec: """{
        //                 "files": [{
        //                     "pattern": "project-${BUILD_NUMBER}/*",
        //                     "target": "${ARTIFACTORY_REPO}/project-${BUILD_NUMBER}/",
        //                     "flat": "true"
        //                 }]
        //             }""", buildInfo: buildInfo

        //             // Publish build info to Artifactory
        //             server.publishBuildInfo buildInfo
        //         }
        //     }
        // }
    
    }

    post {
        success {
            echo 'Tests passed successfully!'
            archiveArtifacts artifacts: '**', excludes: 'venv/** , .env', onlyIfSuccessful: true, fingerprint: true


        }

        failure {
            echo 'Tests failed!'
        }
    }
}
