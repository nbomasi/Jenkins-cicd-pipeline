pipeline {
    agent any

    tools {
        maven "maven3"
        jdk "OracleJDK8"
    }
    stages {
        stage('Fetch code') {
            steps {
                git branch: 'main', url: 'https://github.com/nbomasi/vprofile-project-boma.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Archiving artifacts now.'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        } 

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
             steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml

                   '''
                }   
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }    
        stage('Nexus-Artifact-upload') {
            steps {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '172.31.10.222:8081',
                        groupId: 'QA',
                        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                        repository: 'boma-vprofile-repo',
                        credentialsId: 'nexus-login',
                        artifacts: [
                            [artifactId: 'boma-vprofile',
                            classifier: '',
                            file: 'target/vprofile-v2.war',
                            type: 'war']
                        ]
                    )
            }
        }
        }   
       
}