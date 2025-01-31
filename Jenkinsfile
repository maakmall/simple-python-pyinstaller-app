node {
    checkout scm
    
    def pythonImage = 'python:3.9'
    def pytestImage = 'qnib/pytest'

    stage('Build') {
        docker.image(pythonImage).inside {
            echo 'Compiling Python files...'
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage('Test') {
        docker.image(pytestImage).inside {
            echo 'Running unit tests...'
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }

        junit 'test-reports/results.xml'
    }

    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
    }

    stage('Deploy') {
        docker.image(pythonImage).inside('--user root') {
            echo 'Building Python application...'
            sh 'pip install pyinstaller'
            sh 'pyinstaller --onefile sources/add2vals.py'
        }

        echo 'Archiving artifacts...'
        archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true

        sleep(time: 1, unit: 'MINUTES')
    }
}
