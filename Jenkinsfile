node(){
    withDockerContainer('python:3.8-alpine'){
        stage('Build') {
            checkout scm
            sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
    }
    stage('Manual Approval'){
        input message: 'Sudah selesai menggunakan React App? (Klik "Proceed" untuk mengakhiri)'
    }
    stage('Deploy'){
        withEnv(['VOLUME=$(pwd)/sources:/src', 'IMAGE=cdrx/pyinstaller-linux:python3']) {
            dir(path: env.BUILD_ID) {
                unstash name: 'compiled-results'
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                echo 'Kriteria 3, tunggu 1 menit...'
                sh 'sleep 60'
                archiveArtifacts "sources/dist/add2vals"
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            }
        }
    }
}