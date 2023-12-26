node(){
    stage('Build') {
        sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
        stash(name: 'compiled-results', includes: 'sources/*.py*')
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
    }
    stage('Deliver') {
        dir(path: env.BUILD_ID) {
           unstash name: 'compiled-results'
           sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        }

        archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
    }
}