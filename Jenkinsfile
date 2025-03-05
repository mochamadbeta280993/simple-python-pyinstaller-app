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
        docker.image('cdrx/pyinstaller-linux:python2').inside('--entrypoint=""') {
            sh '''
                python -m ensurepip --default-pip || curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python get-pip.py
                pyinstaller --onefile sources/add2vals.py
            '''
		}
        archiveArtifacts artifacts: 'dist/add2vals'
    }
}