node {
    checkout scm
    
    def pythonImage = 'python:2-alpine'
    def pytestImage = 'qnib/pytest'
    def pyinstallerImage = 'cdrx/pyinstaller-linux:python2'

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
        try {
            dir(env.BUILD_ID) {
                unstash(name: 'compiled-results')

                docker.image(pyinstallerImage).inside {
                    echo 'Running PyInstaller...'
                    sh "pyinstaller -F add2vals.py"
                }

                archiveArtifacts artifacts: "sources/dist/add2vals", fingerprint: true
            }

            sleep(time: 1, unit: 'MINUTES')
        } catch (Exception e) {
            echo "Error during deploy: ${e.getMessage()}"
        }
    }
}
