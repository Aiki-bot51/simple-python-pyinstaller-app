node(){
    stage('Build') {
        sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
        stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    stage('Test') {
        sh 'py.test sources/test_calc.py'
        }
}