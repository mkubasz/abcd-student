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
                        -v /Users/mkubaszek/Projects/abcd/abcd-student/.zap:/zap/wrk/:rw \
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
                    archiveArtifacts artifacts: 'results/*.html, results/*.xml', allowEmptyArchive: true
                }
            }
        }
        stage('OSV-Scanner') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    if ! command -v osv-scanner &> /dev/null; then
                        LATEST_TAG=$(curl -s https://api.github.com/repos/google/osv-scanner/releases/latest | grep "tag_name" | cut -d '"' -f 4)
                        curl -sSL https://github.com/google/osv-scanner/releases/download/$LATEST_TAG/osv-scanner-linux-amd64 -o /usr/local/bin/osv-scanner
                        chmod +x /usr/local/bin/osv-scanner
                    fi
                '''
            }
            post {
                always {
                    sh '''
                        osv-scanner -r package-lock.json --format json > results/osv-response.json || true
                    '''
                    archiveArtifacts artifacts: 'results/osv-response.json', allowEmptyArchive: true
                }
            }
        }
        stage('SAST Scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin
                    trufflehog git file://. --only-verified --json > results/trufflehog_report.json|| true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results/trufflehog_report.json', allowEmptyArchive: true
                }
            }
        }
        stage('Semgrep Scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    if ! command -v semgrep &> /dev/null; then
                        pip install semgrep
                    fi
                '''
                sh '''
                    semgrep --config auto --json > results/semgrep_report.json || true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results/semgrep_report.json', allowEmptyArchive: true
                }
            }
        }
    }
}