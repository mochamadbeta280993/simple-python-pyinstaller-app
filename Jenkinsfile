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
        docker.image('cdrx/pyinstaller-linux:python2').inside('-u root --entrypoint=""') {
            sh '''
                python --version
            
                sed -i 's/archive.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
                apt-get update

                apt-get install -y locales python2.7 python2.7-minimal python2.7-dev
                locale-gen en_US.UTF-8

                curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
                python get-pip.py --no-cache-dir --disable-pip-version-check pip==20.3.4
                
                pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org pyinstaller
                pyinstaller --onefile sources/add2vals.py
            '''
		}
        archiveArtifacts artifacts: 'dist/add2vals'
    }
}