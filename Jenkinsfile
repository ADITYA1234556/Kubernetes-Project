pipeline {
    agent any

    tools{
        jdk 'jdk17'
        maven 'maven3'
    }

    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/ADITYA1234556/Kubernetes-Project.git'
            }
        }
        stage('Compile source code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test the code') {
            steps {
                sh "mvn test"
            }
        }
        stage('Dependency and filesystem Scan using trivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                script {
                    def retries = 3
                    def delay = 60
                    def result = null

                    for (int i = 0; i < retries; i++) {
                        result = waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                        if (result.status == 'OK') {
                            echo "Quality Gate Passed"
                            break
                        } else if (result.status != 'IN_PROGRESS') {
                            error "Quality Gate failed: ${result.status}"
                        }
                        echo "Quality Gate in progress... retrying in ${delay} seconds"
                        sleep(delay)
                    }

                    if (result.status == 'IN_PROGRESS') {
                        error "Quality Gate still stuck after ${retries} retries"
                    }
                }
            }
        }
        stage('Build stage') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Publish Artifacts to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests"
                }
            }
        }
        stage('Build and Tag docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhubcreds', toolName: 'docker') {
                        sh "docker build -t adityahub2255/boardgame:latest ."
                    }
                }
            }
        }
        stage('Docker images scan') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html adityahub2255/boardgame:latest"
            }
        }
        stage('Push docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhubcreds', toolName: 'docker') {
                        sh "docker push adityahub2255/boardgame:latest"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.20.145:6443') {
                    sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
        stage('Check the deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.20.145:6443') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'adityanavaneethan98@gmail.com',
                from: 'adityanavaneethan98@gmail.com',
                replyTo: 'adityanavaneethan98@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
            }
        }
    }
}
