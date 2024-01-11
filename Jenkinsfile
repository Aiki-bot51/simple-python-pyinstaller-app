node(){
    stage('Build') {
        checkout scm
        sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
        stash(name: 'compiled-results', includes: 'sources/*.py*')
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
        }
    }
//    stage('Manual Approval'){
//        input message: 'Apakah hasil sudah OK? (Klik "Proceed" untuk Deploy)'
//    }
    stage('Deploy') {
        withCredentials([string(credentialsId: 'vercel-credentials', variable: 'VERCEL_TOKEN')]) {
            withEnv(['VOLUME=$(pwd)/sources:/src', 'IMAGE=cdrx/pyinstaller-linux:python3']) {
                dir(path: env.BUILD_ID) {
                    unstash name: 'compiled-results'

                    // Create the sources/dist directory
                    sh 'mkdir -p sources/dist'

                    // Run PyInstaller to build add2vals.py in sources/dist
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py --distpath sources/dist'"

                    echo 'Kriteria 3, tunggu 1 menit...'
                    sh 'sleep 60'

                    // Debugging: List contents of the current directory
                    sh 'ls -la'

                    // Debugging: List contents of the dist directory
                    sh 'ls -la sources/dist'

                    // Archive artifacts
                    archiveArtifacts "sources/dist/*"

                    // Deploy only add2vals.py to Vercel using Vercel CLI
                    dir("sources/dist") {
                        // Debugging: List contents of the current directory
                        sh 'ls -la'

                        // Ensure proper usage of add2vals.py
                        sh "vercel --token \$VERCEL_TOKEN --prod --yes add2vals"
                    }
                }
            }
        }
    }
}