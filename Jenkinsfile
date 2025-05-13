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
                    git credentialsId: 'github-token', url: 'https://github.com/mkubasz/abcd-student', branch: 'main'
                }
            }
        }
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    docker rm -f juice-shop || true
                    docker rm -f zap || true
                    
                    # Start the Juice Shop container
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh 'echo "Setting permissions for ${WORKSPACE}/zap" && chmod -R 777 ${WORKSPACE}/zap || echo "Failed to chmod ${WORKSPACE}/zap"'
                sh '''
                    # Run ZAP scan
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/zap/passive.yaml:/opt/custom_zap_config/my_passive_config.yaml:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "
                        mkdir -p /opt/custom_zap_config && echo 'Checking /opt/custom_zap_config:' && ls -la /opt/custom_zap_config/; 
                        echo 'Checking /zap/wrk:' && ls -la /zap/wrk/; 
                        zap.sh -cmd -addonupdate; 
                        zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /opt/custom_zap_config/my_passive_config.yaml" \
                        || true
                '''
            }
            post {
                always {
                    sh '''
                        # Copy reports
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                        docker stop zap juice-shop || true
                        docker rm zap || true
                        docker rm juice-shop || true
                    '''
                }
            }
        }
    }
}