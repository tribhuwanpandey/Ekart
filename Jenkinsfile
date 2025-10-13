pipeline {
    agent any

    environment {
        NVD_API_KEY = credentials('nvd-api-key') // Secret text
        DOCKER_HUB_PASSWORD = credentials('tp109') // Secret text
    }

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tribhuwanpandey/Ekart.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Unit Tests') {
            steps {
                // RUN tests instead of skipping them!
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-token') {
                    script {
                        def scannerHome = tool 'sonar-scanner'
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=EKART \
                            -Dsonar.projectName=EKART \
                            -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                script {
                    dependencyCheck additionalArguments: "--nvdApiKey=${NVD_API_KEY}",
                                    odcInstallation: 'DC' // Make sure this matches your Jenkins tool config
                }
            }
        }

        stage('Build Package') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven',
                          jdk: 'jdk-17',
                          maven: 'maven3',
                          mavenSettingsConfig: '',
                          traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                sh 'docker build -t tp109/ekart:latest -f docker/Dockerfile .'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u tp109 --password-stdin"
                    sh 'docker push tp109/ekart:latest'
                }
            }
        }

        stage('Configure EKS and Kubectl') {
            steps {
                sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deploymentservice.yml'
            }
        }
    }
}
