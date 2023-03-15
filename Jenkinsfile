pipeline {
    agent { label 'UBUNTU_NODE2'}
//    triggers { pollSCM('* * * * *')}
    stages {
        stage('vcs'){
            steps {
                git url: 'https://github.com/srikanthvelma-jenkins/spring-petclinic.git',
                    branch: 'rel_v1.0'
            }
        }
        stage('build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('postbuild') {
            steps {
                archiveArtifacts artifacts: '**/target/spring-petclinic-3.0.0-SNAPSHOT.jar'
                                 junit '**/surefire-reports/TEST-*.xml'
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
        stage('copy build') {
            steps{
                sh "mkdir -p /tmp/archive/${JOB_NAME}/${BUILD_ID} && cp ./target/spring-petclinic-*.jar /tmp/archive/${JOB_NAME}/${BUILD_ID}/"
                sh "aws s3 sync /tmp/archive/${JOB_NAME}/${BUILD_ID} s3://srikanthcicd --acl public-read-write"
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