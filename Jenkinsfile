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
        def volume = "${pwd()}/sources:/src"
        def image = 'cdrx/pyinstaller-linux:python3'

        withEnv(["volume=${volume}", "image=${image}"]) {
            dir(path: env.BUILD_ID) {
                unstash name: 'compiled-results'
                sh "docker run --rm -v "${volume}""${image}" 'pyinstaller -F add2vals.py'"
            }
        }

        post {
            success {
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                sh "docker run --rm -v "${volume}""${image}" 'rm -rf build dist'"
            }
        }
    }
}