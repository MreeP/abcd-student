pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Fetch the code') {
            steps {
                echo 'Cloning the repository...'
                
                script {
                    cleanWs()
                    git credentialsId: 'github-personal-access-token', url: 'https://github.com/MreeP/abcd-student', branch: 'main'
                }
            }
        }
        stage('Setup environment') {
            steps {
                echo 'Setting up the environment...'
            
                sh 'mkdir -p results'

                // echo 'Starting the container...'
                // 
                // sh '''
                //     docker run \
                //         --name juice \
                //         -d \
                //         -p 3000:3000 \
                //         bkimminich/juice-shop
                // '''
            }
        }
        stage('Run SCA scan - osv scanner') {
            steps {    
                echo 'Starting the osv scan...'
                sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/osv-scanner-output.json'
            }
            post {
                always {
                    //
                }
            }
        }
        // stage('Run DAST scan - ZAP') {
        //     steps {    
        //         echo 'Starting the zap container...'
                
        //         sh '''
        //             docker run \
        //                 --name zap \
        //                 --add-host=host.docker.internal:host-gateway \
        //                 -v /Users/kk/Documents/Courses/ABCDevSecOps/abcd-student/.zap:/zap/wrk/:rw \
        //                 -t ghcr.io/zaproxy/zaproxy:stable bash \
        //                 -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
        //                 || true
        //         '''
        //     }
        //     post {
        //         always {
        //             echo 'Copying the results and removing the container...'
                    
        //             sh '''
        //                 docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
        //                 docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
        //                 docker stop zap
        //                 docker rm zap
        //             '''
        //         }
        //     }
        // }
        stage('Cleanup') {
            steps {
                echo 'Stopping and removing the containers'
                
                sh '''
                    docker stop juice
                    docker rm juice
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            // defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'kajetan.kucharski@secawa.com')
        }
    }
}
