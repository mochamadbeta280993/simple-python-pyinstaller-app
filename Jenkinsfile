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
                    sh 'git push https://heroku:$HEROKU_API_KEY@git.heroku.com/submission-cicd-pipeline-mba.git main &'
                    sleep 60
                    sh 'heroku run "find / -type f -name 'add2vals' 2>/dev/null" -a submission-cicd-pipeline-mba'

                    // // Create an artifacts directory if not exists
                    // sh 'mkdir -p artifacts'

                    // // Encode the binary to base64 (avoid corruption in transfer) and transfer the binary safely
                    // sh 'heroku run "base64 /app/dist/add2vals" -a submission-cicd-pipeline-mba | tee artifacts/add2vals.b64'

                    // sh '''
                    //     ls -lh artifacts/add2vals.b64
                    //     cat artifacts/add2vals.b64 | head -n 10
                    // '''

                    // // Decode it back to binary in Jenkins
                    // sh 'base64 -d artifacts/add2vals.b64 > artifacts/add2vals'
                    // sh 'rm artifacts/add2vals.b64'
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