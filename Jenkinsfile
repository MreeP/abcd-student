pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('GitHub') {
            steps {
                echo 'Cloning the repository...'
                
                script {
                    cleanWs()
                    git credentialsId: 'github-personal-access-token', url: 'https://github.com/MreeP/abcd-student', branch: 'main'
                }
            }
        }
        stage('Setup') {
            echo 'Setting up the environment...'
            
            sh 'mkdir -p results'
        }
        stage('ZAP') {
            steps {
                echo 'Starting the container...'
                
                sh '''
                    docker run \
                        --name juice \
                        -d \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                        
                    sleep 5
                '''
                
                echo 'Starting the zap container...'
                
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
                    echo 'Copying the results...'
                    
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                    '''
                }
            }
        }
        stage('Cleanup') {
            steps {
                echo 'Stopping the containers'
                
                sh '''
                    docker stop zap
                    docker stop juice
                '''

                echo 'Removing unused containers'
                
                sh '''
                    docker rm zap
                    docker rm juice
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
