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
                sh 'mkdir -p results/ zap/reports/'
                sh '''
                    # Clean up any existing containers with these names
                    docker rm -f juice-shop || true
                    docker rm -f zap || true
                    
                    # Start the Juice Shop container
                    docker run --name juice-shop -d --rm \\
                        -p 3000:3000 \\
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    # Run ZAP scan
                    docker run --name zap \\
                        --add-host=host.docker.internal:host-gateway \\
                        -v ${WORKSPACE}/zap:/zap/wrk/:rw \\
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \\
                        "mkdir -p /zap/wrk/reports && \\
                        zap.sh -cmd -addonupdate && \\
                        zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta && \\
                        zap.sh -cmd -autorun /zap/wrk/passive.yaml" \\
                        || true
                '''
            }
            post {
                always {
                    sh '''
                        # Verify if the container is still running
                        if docker ps | grep -q zap; then
                            # List files in the reports directory to debug
                            docker exec zap ls -la /zap/wrk/reports || true
                            
                            # Copy reports
                            docker cp zap:/zap/wrk/reports/. ${WORKSPACE}/results/ || true
                        fi
                        
                        # Clean up containers
                        docker stop zap juice-shop || true
                        docker rm zap || true
                    '''
                }
            }
        }
    }
}