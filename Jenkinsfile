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
                curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
                apt-get update && apt-get install -y locales
                locale-gen en_US.UTF-8
                python get-pip.py
                pip install pyinstaller
                pyinstaller --onefile sources/add2vals.py
            '''
		}
        archiveArtifacts artifacts: 'dist/add2vals'
    }
}