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
}