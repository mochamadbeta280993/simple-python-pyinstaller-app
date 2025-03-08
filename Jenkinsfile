node {
    // Preparation stage: Set up workspace and ensure correct branch
    stage('Preparation') {
        // Ensure Jenkins has proper ownership and permissions on the workspace
        sh 'sudo chown -R jenkins:jenkins /var/jenkins_home/workspace'
        sh 'sudo chmod -R 755 /var/jenkins_home/workspace'
        
        // Check out the source code from the repository
        checkout scm

        // Ensure the branch is 'main'; if not, switch to 'main'
        sh '''
            CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
            if [ "$CURRENT_BRANCH" != "main" ]; then
                COMMIT_HASH=$(git rev-parse HEAD)
                git checkout -B main $COMMIT_HASH
            fi
        '''
        
        // Display the available branches and list all files in the workspace
        sh 'git branch -a'
        sh 'ls -R'
    }

    // Build stage: Compile Python source files
    stage('Build') {
        // Run the build process inside a Python 2 Alpine Docker container
        docker.image('python:2-alpine').inside('-u root') {
            // Compile Python files to check for syntax errors
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    // Test stage: Run unit tests
    stage('Test') {
        try {
            // Run tests inside a Docker container with pytest installed
            docker.image('qnib/pytest').inside {
                // Execute the test script and generate a JUnit XML report
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        } catch (err) {
            // If tests fail, report an error and fail the pipeline
            error "Test failed: ${err}"
        } finally {
            // Always publish test results, even if tests fail
            junit 'test-reports/results.xml'
        }
    }

    // Deployment stage: Deploy the application to Heroku
    stage('Deploy') {
        // Variable to track deployment success
        def deploySuccessful = false
        try {
            // Run deployment process inside a Python 3.9 Docker container
            docker.image('python:3.9').inside('-u root') {
                // Use credentials stored in Jenkins to authenticate with Heroku
                withCredentials([string(credentialsId: 'HEROKU_API_KEY', variable: 'HEROKU_API_KEY')]) {
                    // Install Heroku CLI
                    sh 'curl https://cli-assets.heroku.com/install.sh | sh'
                    
                    // Change workspace ownership to avoid permission issues
                    sh 'chown -R $(id -u):$(id -g) "$WORKSPACE"'

                    // Add Heroku as a remote repository
                    sh 'heroku git:remote -a submission-cicd-pipeline-mba'

                    // Set OpenSSL legacy mode before pushing to Heroku
                    sh 'heroku config:set NODE_OPTIONS=--openssl-legacy-provider -a submission-cicd-pipeline-mba'

                    // Push code to Heroku repository for deployment
                    sh 'git push https://heroku:$HEROKU_API_KEY@git.heroku.com/submission-cicd-pipeline-mba.git main'

                    // Fetch the latest release log from Heroku
                    def releaseLog = sh(script: "heroku releases -a submission-cicd-pipeline-mba --json | jq -r '.[0].description'", returnStdout: true).trim()
                    
                    // Print the log
                    sh 'echo "Latest Release Log:" && echo "${releaseLog}" | tail -n 100'

                    // // Run the Heroku logs command and store output in deploy.log
                    // def deployLog = sh(script: '''
                    //     set -o pipefail
                    //     heroku logs --source release -a submission-cicd-pipeline-mba | awk '/BASE64_START/{flag=1;next}/BASE64_END/{flag=0}flag' > add2vals.b64
                    // ''', returnStdout: true).trim()
                }
            }
            // Mark deployment as successful
            deploySuccessful = true
        } catch (err) {
            // If deployment fails, report an error and fail the pipeline
            error "Deploy failed: ${err}"
        } finally {
            // If deployment was successful, archive the build artifacts
            if (deploySuccessful) {
                // Verify if the downloaded file is valid
                // sh 'ls -lh artifacts/add2vals'
                // sh 'file artifacts/add2vals'
                // sh 'sudo chmod +x artifacts/add2vals'
                
                // Archive the retrieved binary so it appears in Jenkins artifacts
                // archiveArtifacts 'artifacts/add2vals'
            }
        }
    }
}