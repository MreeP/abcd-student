pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-personal-access-token', url: 'https://github.com/MreeP/abcd-student', branch: 'main'
                }
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }
        stage('Passive scan - ZAP') {
            steps {
                sh '''
                    docker run \
                        --name juice \
                        -d \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                        
                    sleep 5
                '''
                sh '''
                    docker run \
                        --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /Users/kk/Documents/Courses/ABCDevSecOps/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash \
                        -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                        || true
                    
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        
                        docker stop zap
                        
                        docker stop juice

                        docker rm zap

                        docker rm juice
                    '''
                }
            }
        }
    }
}
