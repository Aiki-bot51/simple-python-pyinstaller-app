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
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} /bin/bash -c 'rm -rf build dist'"
                    //sh sleep 60
                    echo 'Kriteria 3, tunggu 1 menit...'
                    archiveArtifacts "sources/dist/add2vals"

                    script {
                        // Deploy to Vercel
                        def vercelDeployOutput = sh(script: "vercel --token=${VERCEL_TOKEN} --prod $(pwd)/sources/dist/add2vals", returnStatus: true)
                        
                        // Check if the deployment already exists and use redeploy if true
                        if (vercelDeployOutput == 0) {
                            sh "vercel --token=${VERCEL_TOKEN} --prod --confirm $(pwd)/sources/dist/add2vals"
                        }
                    }
                }
            }
        }
    }
}