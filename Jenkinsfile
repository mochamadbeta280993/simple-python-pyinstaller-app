node {
    // stage('Build') {
    //     checkout scm        
    //     docker.image('python:2-alpine').inside {
    //         sh 'python -m py_compile sources/add2vals.py sources/calc.py'
    //     }
    // }

    // stage('Test') {
    //     docker.image('qnib/pytest').inside {
    //         sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
    //     }
    //     junit 'test-reports/results.xml'
    // }

    def buildSuccessful = false
    try {
        stage('Deliver') {
            docker.image('python:3.9').inside('-u root') {
                withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {
                    sh 'curl https://cli-assets.heroku.com/install.sh | sh'
                    
                    // Change ownership of workspace
                    sh 'chown -R $(id -u):$(id -g) "$WORKSPACE"'

//                    sh 'git checkout main || git checkout -b main origin/main'

                    sh 'heroku git:remote -a submission-cicd-pipeline-mba'

                    // Set OpenSSL legacy mode before pushing
                    sh 'heroku config:set NODE_OPTIONS=--openssl-legacy-provider -a submission-cicd-pipeline-mba'

                    // Push to Heroku
                    sh 'git push https://heroku:$HEROKU_API_KEY@git.heroku.com/submission-cicd-pipeline-mba.git main'

                    // sh 'pip install pyinstaller'

                    // sh 'pyinstaller --onefile sources/add2vals.py'
                }
            }
            buildSuccessful = true
        }
    } catch (err) {
        error "Build failed: ${err}"
    } finally {
        if (buildSuccessful) {
            archiveArtifacts 'dist/add2vals'
        }
    }
}