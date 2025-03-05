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

    stage('Deliver') {
        sh '''
        echo "Checking Docker version and user..."
        docker version
        whoami
        id
        echo "Checking Jenkins workspace..."
        ls -lah $WORKSPACE
        echo "Checking running containers before execution..."
        docker ps -a
        echo "Attempting to start PyInstaller container..."
        docker run --rm --entrypoint /bin/sh cdrx/pyinstaller-linux:python2 -c "echo Container started successfully"
        echo "Checking running containers after execution..."
        docker ps -a
        echo "Debugging complete."
        '''
//        docker.image('cdrx/pyinstaller-linux:python3').inside('-v $WORKSPACE:/workspace --entrypoint /bin/sh') {
//            sh 'pyinstaller --onefile sources/add2vals.py'
//        }       
//        archiveArtifacts artifacts: 'dist/add2vals'
    }
}