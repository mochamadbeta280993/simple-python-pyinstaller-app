node {
    stage('Build') {
        // Pull the latest code from SCM
        checkout scm
        // Run inside python:2-alpine Docker container
        docker.image('python:2-alpine').inside {
            // Compile Python sources
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage('Test') {
        // Run tests inside qnib/pytest Docker container
        docker.image('qnib/pytest').inside {
            // Execute pytest, output JUnit XML
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        // Publish JUnit test results
        junit 'test-reports/results.xml'
    }

    stage('Deliver') {
        // Build final executable inside cdrx/pyinstaller-linux:python2
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile sources/add2vals.py'
        }
        // Archive the artifact so it can be downloaded from Jenkins
        archiveArtifacts artifacts: 'dist/add2vals'
    }
}