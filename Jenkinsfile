pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven "Maven3"
    }

    environment {
        SCANNER_HOME=tool'sonar-scanner'
    }

    stages {
        stage('GitCheckout') {
            steps {
                git 'https://github.com/rajsumanth19/Mission.git'
            }
        }

        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                // Run the tests
                sh 'mvn test'
            }
        }

        stage('Trivy Scan File System') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Mission \
                        -Dsonar.projectName=Mission \
                        -Dsonar.java.binaries=.'''

                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Deploy Artifacts To Nexus'){
            steps { 
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 
                'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                } 
            }
        }    

        stage('Build&Tag DockerImage') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t raja3546/mission:latest ."
                    }
                }
            }
        }

        stage('Trivy Scan Image') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html raja3546/mission:latest"
            }
        }

        stage('Publish DockerImage') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker_token', toolName: 'docker') {
                        sh "docker push raja3546/mission:latest"
                   }
                }
            }    
        }
    }   

}
    
