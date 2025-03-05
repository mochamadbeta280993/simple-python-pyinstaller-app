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
        echo "Starting container for debugging..."
        docker run -d --name pyinstaller_debug cdrx/pyinstaller-linux:python2 tail -f /dev/null
        echo "Listing running containers..."
        docker ps -a
        echo "Entering the container..."
        docker exec -it pyinstaller_debug /bin/sh
        '''
//        docker.image('cdrx/pyinstaller-linux:python3').inside('-v $WORKSPACE:/workspace --entrypoint /bin/sh') {
//            sh 'pyinstaller --onefile sources/add2vals.py'
//        }       
//        archiveArtifacts artifacts: 'dist/add2vals'
    }
}