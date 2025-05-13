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
                    
                    # Copy the passive.yaml to a temporary file that we know we can mount
                    cat zap/passive.yaml > passive-scan.yaml
                    
                    # Set permissions
                    chmod 777 passive-scan.yaml
                    chmod -R 777 zap/ results/
                    
                    # List files to verify
                    echo "Files in workspace:"
                    ls -la
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
                    # Run ZAP scan with directly mounted config file
                    docker run --name zap \\
                        --add-host=host.docker.internal:host-gateway \\
                        -v ${WORKSPACE}/passive-scan.yaml:/zap/passive-scan.yaml:ro \\
                        -v ${WORKSPACE}/zap/reports:/zap/wrk/reports:rw \\
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \\
                        "echo 'Contents of root directory:' && \\
                        ls -la /zap/ && \\
                        echo 'Verifying passive-scan.yaml:' && \\
                        cat /zap/passive-scan.yaml && \\
                        echo 'Installing ZAP add-ons...' && \\
                        zap.sh -cmd -addonupdate && \\
                        zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta && \\
                        echo 'Starting ZAP scan...' && \\
                        zap.sh -cmd -autorun /zap/passive-scan.yaml" \\
                        || echo "ZAP scan failed but continuing the pipeline"
                '''
            }
            post {
                always {
                    sh '''
                        # Verify if the container is still running
                        if docker ps | grep -q zap; then
                            # List files in the reports directory to debug
                            echo "Checking for report files:"
                            docker exec zap ls -la /zap/wrk/reports || true
                            
                            # Copy reports - try both potential locations
                            echo "Copying report files to results directory:"
                            docker cp zap:/zap/wrk/reports/. ${WORKSPACE}/results/ || true
                            
                            # In case reports were generated in a different directory
                            docker exec zap find /zap -name "*.html" -o -name "*.xml" | sort || true
                        fi
                        
                        # List the content of results directory
                        echo "Results directory contents:"
                        ls -la ${WORKSPACE}/results/ || true
                        
                        # Clean up containers
                        echo "Cleaning up containers:"
                        docker stop zap juice-shop || true
                        docker rm zap || true
                    '''
                    
                    archiveArtifacts artifacts: 'results/*.html,results/*.xml', allowEmptyArchive: true
                }
            }
        }
    }
}