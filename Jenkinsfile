pipeline {
    agent { label 'UBUNTU_NODE2'}
//    triggers { pollSCM('* * * * *')}
    stages {
        stage('vcs'){
            steps {
                git url: 'https://github.com/srikanthvelma-jenkins/spring-petclinic.git',
                    branch: 'rel_v2.0'
            }
        }
        stage('Artifactory config') {
            steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: 'https://srikanthvelma.jfrog.io/artifactory',
                    credentialsId: 'JFROG_KEY'
                )
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: 'libs-release',
                    snapshotRepo: 'libs-snapshot'
                )    
            }
        }
        stage('build') {
            steps {
                rtMavenRun (
                    tool: 'MAVEN_17',
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER"
                )
                rtPublishBuildInfo (
                    serverId: "ARTIFACTORY_SERVER"
                )
            }
        }
        stage('sonar analysis') {
            steps {
                 withSonarQubeEnv('SONARQUBE_CLOUD') {
                    sh 'mvn clean verify sonar:sonar \
                        -Dsonar.organization=springpetclinic57\
                        -Dsonar.projectKey=springpetclinic57_petclinic1'
                }
            }
        }
        stage('archiving artifact & junit results') {
            steps {
                archiveArtifacts artifacts: '**/target/spring-petclinic-*.jar'
                                 junit '**/surefire-reports/TEST-*.xml'
            }
        }
        stage('deploy'){
            agent { label 'UBUNTU_NODE1'}
            steps{
        
                sh 'ansible-playbook -i ./hosts ./spc.yaml'
            }
        }
    }
}