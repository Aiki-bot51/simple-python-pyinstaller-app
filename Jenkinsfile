node(){
    stage('Build') {
        sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
        }
    stage('Test') {
        //image 'qnib/pytest'
        sh 'python3 py.test sources/test_calc.py'
        }
}