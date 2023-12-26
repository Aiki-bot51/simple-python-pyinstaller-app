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
                sh 'docker run -v $(pwd)/sources:/src cdrx/pyinstaller-linux:python2 \'pyinstaller -F add2vals.py\''
                unstash name: 'compiled-results'
                archiveArtifacts "sources/dist/add2vals"
                sh 'docker run -v $(pwd)/sources:/src cdrx/pyinstaller-linux:python2 \'rm -rf build dist\''
            }
        
    }
}