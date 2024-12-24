pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // Corrected spelling here
    }

    stages {
        stage('SCM Checkout') {
            steps {
                // Get code from the GitHub repository
                git ' https://github.com/jiku01/mysecretsanta.git'
            }
        }

        stage('Compile') {
            steps {
                // Compile the code using Maven
                sh "mvn clean compile"
            }
        }

        stage('Sonar Analysis') {
            steps {
                // Run SonarQube analysis
                sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.url= http://18.175.184.217:9000 \
                    -Dsonar.login= squ_a415525476d5db85869f067cf162f09fb0416841\
                    -Dsonar.projectName=Santa \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Santa
                '''
            }
        }

        stage('Code-Build') {
            steps {
                // Build the project using Maven
                sh "mvn clean install"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Use Jenkins credentials to securely login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh ''' 
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            docker build -t santa123 .
                            
                            # Tag image with Jenkins build number as version
                            docker tag santa123 jiku01/santa123:$BUILD_NUMBER

                            # Push the tagged image to Docker Hub
                            docker push jiku01/santa123:$BUILD_NUMBER
                        '''
                    }
                }
            }
        }

        stage('Docker Deployment') {
            steps {
                script {
                    // Deploy the Docker container
                    // Ensure the container is stopped (if already running) and removed
                    sh '''
                        # Stop and remove any previous containers (if running)
                        docker stop santa-container || true
                        docker rm santa-container || true

                        # Run the new container exposing port 8080
                        docker run -d --name santa-container -p 8080:8080 jiku01/santa123:$BUILD_NUMBER
                    '''
                }
            }
        }
    }
}
