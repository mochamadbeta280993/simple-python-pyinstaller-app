node {
    //
    stage('Preparation') {
        //
        sh 'sudo chown -R jenkins:jenkins /var/jenkins_home/workspace'
        sh 'sudo chmod -R 755 /var/jenkins_home/workspace'
        checkout scm

        //
        sh '''
            CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
            if [ "$CURRENT_BRANCH" != "main" ]; then
                COMMIT_HASH=$(git rev-parse HEAD)
                git checkout -B main $COMMIT_HASH
            fi
        '''
        sh 'git branch -a'
        sh 'ls -R'
    }

    //
    stage('Build') {
        //
        docker.image('python:2-alpine').inside('-u root') {
            //
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    //
    stage('Test') {
        try {
            //
            docker.image('qnib/pytest').inside {
                //
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        } catch (err) {
            //
            error "Test failed: ${err}"
        } finally {
            // Always publish test results, even if tests fail
            junit 'test-reports/results.xml'
        }
    }

    //
    def deploySuccessful = false
    try {
        //
        stage('Deploy') {
            //
            docker.image('python:3.9').inside('-u root') {
                //
                withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {
                    //
                    sh 'curl https://cli-assets.heroku.com/install.sh | sh'
                    
                    // Change ownership of workspace
                    sh 'chown -R $(id -u):$(id -g) "$WORKSPACE"'

                    //
                    sh 'heroku git:remote -a submission-cicd-pipeline-mba'

                    // Set OpenSSL legacy mode before pushing
                    sh 'heroku config:set NODE_OPTIONS=--openssl-legacy-provider -a submission-cicd-pipeline-mba'

                    // Push to Heroku
                    sh 'git push https://heroku:$HEROKU_API_KEY@git.heroku.com/submission-cicd-pipeline-mba.git main'
                }
            }
            //
            deploySuccessful = true
        }
    } catch (err) {
        //
        error "Deploy failed: ${err}"
    } finally {
        //
        if (deploySuccessful) {
            //
            archiveArtifacts 'dist/add2vals'
        }
    }
}