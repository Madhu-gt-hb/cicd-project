def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    tools {
        maven "MAVEN3"
        jdk "OracleJDK21"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'OneplusNew@12'
        RELEASE_REPO = 'vproapp'
        CENTRAL_REPO = 'cicd-maven-central'

        NEXUS_IP = '18.61.165.27'
        NEXUS_PORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'Nexuslogin'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests clean install'
            }
            post {
                success {
                    echo 'Archiving WAR file'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=cicd-project \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.tests=src/test/java \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload Artifact') {
            steps {
                script {

                    // SAFE VERSION (NO SPACES / TIMESTAMP ISSUES)
                    def VERSION = new Date().format("yyyyMMdd-HHmmss")

                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",

                        groupId: 'QA',
                        version: "1.0-${VERSION}",

                        repository: "${RELEASE_REPO}",
                        credentialsId: "${NEXUS_LOGIN}",

                        artifacts: [
                            [
                                artifactId: 'vproapp',
                                classifier: '',
                                file: 'target/vprofile-v2.war',
                                type: 'war'
                            ]
                        ]
                    )
                }
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications'

            slackSend channel: '#cicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\nMore info at: ${env.BUILD_URL}"
        }
    }
}
