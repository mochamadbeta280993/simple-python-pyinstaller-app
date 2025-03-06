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
                // Change ownership of workspace
                sh 'chown -R $(id -u):$(id -g) "$WORKSPACE"'

                sh 'git remote set-url origin file://$WORKSPACE'
                sh 'git fetch --all'

                sh 'git branch -a'

                sh 'git status'

                // sh '''
                //     git checkout main || git checkout -b main origin/master
                //     git pull origin master
                // '''

                sh 'git checkout main || git checkout -b main'

                sh 'git remote -v'

                sh 'git status'

                sh 'git branch -a'

                sh 'cat Jenkinsfile'
                
                sh '''
                    pip install pyinstaller
                    pyinstaller --onefile sources/add2vals.py
                '''
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