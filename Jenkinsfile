node {
    stage('Build') {
        checkout scm        
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        junit 'test-reports/results.xml'
    }

    def buildSuccessful = false
    try {
        stage('Deliver') {
            docker.image('python:3.9').inside('-u root') {
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
            stage('Post-Success') {
                archiveArtifacts 'dist/add2vals'
            }
        }
    }
}