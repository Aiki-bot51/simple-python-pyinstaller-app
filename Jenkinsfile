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
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                    echo 'Kriteria 3, tunggu 1 menit...'
                    archiveArtifacts "sources/dist/add2vals"

                    script {
                        echo "Current working directory: ${pwd()}"
                        // Navigate to the deployment directory
                        dir("sources/dist") {
                            echo "Changed directory to: ${pwd()}"
                            // Deploy to Vercel from the directory
                            sh "vercel --token=\${VERCEL_TOKEN} --prod -y"
        
                            // Check if the deployment already exists and use redeploy if true
                            sh "vercel --token=\${VERCEL_TOKEN} --prod --confirm -y"
                        }
                    }
                }
            }
        }
    }
}