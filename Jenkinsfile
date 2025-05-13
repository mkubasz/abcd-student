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
                sh '''
                    # Create directories with proper permissions
                    mkdir -p results/
                    mkdir -p zap/reports/
                    
                    # Set permissions
                    chmod -R 777 zap/ results/
                    
                    # List the zap directory to verify files
                    echo "ZAP directory contents:"
                    ls -la zap/
                '''
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
                        -v ${WORKSPACE}/zap/reports:/zap/wrk/reports:rw \\
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \\
                        "echo 'Contents of /zap/wrk/:' && \\
                        ls -la /zap/wrk/ && \\
                        if [ -f /zap/wrk/passive.yaml ]; then \\
                            echo 'Passive scan configuration file found' && \\
                            zap.sh -cmd -addonupdate && \\
                            zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta && \\
                            zap.sh -cmd -autorun /zap/wrk/passive.yaml; \\
                        else \\
                            echo 'ERROR: passive.yaml not found!' && \\
                            exit 1; \\
                        fi" \\
                        || echo "ZAP scan failed but continuing the pipeline"
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