pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        SONAR_SCANNER_HOME = tool 'sonar_scanner'
        SONARQUBE_SERVER = 'sonarqubeserver'
        NEXUS_CREDENTIAL_ID = 'nexus'
        TOMCAT_CREDENTIAL_ID = 'tomcat'

        NEXUS_URL = 'http://44.197.183.55:8081'
        NEXUS_REPOSITORY = 'sonarqube'
        NEXUS_VERSION = 'nexus3'
        NEXUS_PROTOCOL = 'http'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'feature-1.1',
                    url: 'https://github.com/alsamdevops/sabear_simplecutomerapp.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.binaries=target/classes/ \
                        -Dsonar.junit.reportPaths=target/surefire-reports \
                        -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                        -Dsonar.java.binaries=src/
                    """
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifact = findFiles(glob: "target/*.${pom.packaging}")[0]

                    echo "Found artifact: ${artifact.name} at ${artifact.path}"

                    nexusArtifactUploader(
                        nexusVersion: "${NEXUS_VERSION}",
                        protocol: "${NEXUS_PROTOCOL}",
                        nexusUrl: "${NEXUS_URL}",
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: "${NEXUS_REPOSITORY}",
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: artifact.path,
                             type: pom.packaging],
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: 'pom.xml',
                             type: 'pom']
                        ]
                    )
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifact = findFiles(glob: "target/*.${pom.packaging}")[0]

                    echo "Deploying ${artifact.name} to Tomcat"

                    deploy adapters: [tomcat9(credentialsId: "${TOMCAT_CREDENTIAL_ID}",
                        path: '', 
                        url: 'http://34.200.249.14:8080/manager/text')],
                        contextPath: null,
                        war: artifact.path
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
