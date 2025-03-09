node {
    // Preparation stage: Set up workspace and ensure correct branch
    stage('Preparation') {
        // Ensure Jenkins has proper ownership and permissions on the workspace
        sh 'sudo chown -R jenkins:jenkins /var/jenkins_home/workspace'
        sh 'sudo chmod -R 777 /var/jenkins_home/workspace'
        
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
                script {
                    // Retrieve all available logs
                    def logLines = currentBuild.rawBuild.getLog(Integer.MAX_VALUE)

                    // Extract logs between BASE64_START and BASE64_END
                    def startIndex = -1
                    def endIndex = -1
                    logLines.eachWithIndex { line, index ->
                        if (startIndex == -1 && line.contains("BASE64_START")) {
                            startIndex = index + 1 // Start AFTER BASE64_START
                        }
                        if (line.contains("BASE64_END") && startIndex != -1) {
                            endIndex = index - 1 // Stop BEFORE BASE64_END
                            return
                        }
                    }

                    // Extract logs between BASE64_START and BASE64_END
                    def extractedLogs = []
                    if (startIndex != -1 && endIndex != -1) {
                        extractedLogs = logLines[startIndex..endIndex]
                    } else {
                        echo "BASE64_START or BASE64_END not found in logs!"
                    }

                    // Ensure clean Base64 encoding
                    extractedLogs = extractedLogs.collect { 
                        it.replaceFirst(/^remote:\s*/, "").trim()  // Remove "remote: " and trim spaces
                    }.findAll { it =~ /^[A-Za-z0-9+\/=]+$/ }  // Ensure only valid Base64 characters remain

                    // Save cleaned Base64 data to a file
                    def encodedFile = "${env.WORKSPACE}/encoded_content.b64"
                    writeFile file: encodedFile, text: extractedLogs.join("\n")

                    // Ensure output directory exists before decoding
                    def outputDir = "${env.WORKSPACE}/decoded_files"
                    sh "mkdir -p ${outputDir}"  // Create directory if it doesn't exist

                    // Decode Base64 to create an executable
                    def executableFile = "${outputDir}/add2vals"
                    sh "base64 -d -i ${encodedFile} > ${executableFile}"
                    sh 'sudo chown -R jenkins:jenkins /var/jenkins_home/workspace'
                    sh 'sudo chmod -R 777 /var/jenkins_home/workspace'
                    sh "chmod +x ${executableFile}"  // Make it executable

                    // Archive the executable
                    archiveArtifacts artifacts: 'decoded_files/add2vals'
                }
            }
        }
    }
}