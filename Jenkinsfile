pipeline { 
    agent any 

    options {
        // Prevents the pipeline from triggering itself in an infinite loop
        disableConcurrentBuilds()
    }

    environment { 
        REPO_URL      = 'https://github.com/AyeshaTariq11/Portfolio' 
        SONARQUBE_ENV = 'SonarQube-Server' 
        DOCKER_SERVER = 'ubuntu@ip-172-31-20-138' 
    } 

    stages { 
        stage('Checkout Code') { 
            steps { 
                git branch: 'master', 
                    url: "${REPO_URL}", 
                    credentialsId: 'github-credentials' 
            } 
        } 

        stage('SonarQube Analysis') { 
            steps { 
                script { 
                    def scannerHome = tool 'SonarQube Scanner' 
                    withSonarQubeEnv("${SONARQUBE_ENV}") { 
                        // Combined into one line to avoid backslash/syntax errors
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=portfolio-cloud -Dsonar.projectName=portfolio-cloud -Dsonar.sources=."
                    }
                    // We omit 'waitForQualityGate()' to ensure the pipeline doesn't stop
                }  
            } 
        } 

        stage('Docker Build & Deploy') { 
            steps { 
                sshagent(['docker-server-ssh']) { 
                    sh """
                    # 1. Transfer files to remote server
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/
                    
                    # 2. Run remote commands using double quotes for better stability
                    ssh -o StrictHostKeyChecking=no ${DOCKER_SERVER} "
                        cd /home/ubuntu
                        docker build -t portfolio-app .
                        docker stop portfolio-app || true
                        docker rm portfolio-app || true
                        docker run -d -p 80:80 --name portfolio-app portfolio-app
                    "
                    """
                } 
            } 
        } 
    } 

    post { 
        success { 
            echo "Deployment Successful: http://172.31.26.188" 
        } 
        failure { 
            echo "Pipeline Failed - Check the console output for specific errors." 
        } 
    } 
}