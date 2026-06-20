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
        RELEASE_REPO = 'cicd-release'
        CENTRAL_REPO = 'cicd-maven-central'
        NEXUS_IP = '18.61.165.27'
        NEXUS_PORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Archiving'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
    
        }
    }        
}
