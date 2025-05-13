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
                sh '''
                    # Run ZAP scan
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v $/Users/mkubaszek/Projects/abcd/abcd-student/.zap:/zap/wrk/:rw \
                   -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
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