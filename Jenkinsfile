node(){
    stage('Build') {
        //image 'python:3.12.1-alpine3.19'
        sh 'py_compile sources/add2vals.py sources/calc.py'
        //stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    stage('Test') {
        //image 'qnib/pytest'
        sh 'py.test sources/test_calc.py'
        }
}