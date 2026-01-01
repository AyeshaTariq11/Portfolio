pipeline { 
    agent any 

    options {
        // Sabse important: Ye loop ko rokta hai
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }

    environment { 
        REPO_URL      = 'https://github.com/AyeshaTariq11/Portfolio' 
        SONARQUBE_ENV = 'SonarQube-Server' 
        DOCKER_SERVER = 'ubuntu@ip-172-31-20-138' 
        SONAR_SCANNER_OPTS = "-Xmx512m" 
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
                        // Isko aik hi line mein rakhein taake '\' wala error na aaye
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=portfolio-cloud -Dsonar.projectName=portfolio-cloud -Dsonar.sources=. -Dsonar.exclusions=**/node_modules/**,**/*.png -Dsonar.scm.disabled=true"
                    }
                }  
            } 
        } 

        stage('Docker Build & Deploy') { 
            steps { 
                // Ensure karein ke 'docker-credentials' ka ID Jenkins mein sahi hai
                sshagent(['docker-credentials']) { 
                    sh """
                    scp -o StrictHostKeyChecking=no index.html Dockerfile ${DOCKER_SERVER}:/home/ubuntu/
                    
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
        success { echo "Done!" }
        failure { echo "Failed!" }
    } 
}