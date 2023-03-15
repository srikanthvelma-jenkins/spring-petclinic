Spring Pet Clinic Project Deployment  from Jenkins (CI/CD) through Ansible Playbook
------------------------------------------------------------------------------------
### Tasks
1. Creating a Jenkins pipeline in develop Branch for Day Build 
2. Create a Jenkins Job for merging pull requset into develop branch
3. Creating a Jenkins pipeline in rel_v1.0 Branch for deployment (s3 bucket)
4. Creating a Jenkins pipeline in rel_v2.0 Branch for deployment (jfrog artifactory) 
   

### 1.Creating a Jenkins pipeline in develop Branch for Day Build 
**Pre-requisites**
* for spc - java 17 & maven 3.8 and above required, to installed in the node
* In jenkins , node(UBUNTU_NODE2) to be configured and running
* In Jenkins => Global Tool Config ,we should config tools req with home path
* Tools - java(JDK_17) and maven(MAVEN_17)
  ![preview](images/spcpr1.png)
  ![preview](images/spcpr2.png)
* Create a Jenkins Job => pipeline type and configure as below process
  ![preview](images/spcpr3.png)
  ![preview](images/spcpr4.png)
* Create a Jenkinsfile in our GitHub in the develop branch with stages as below
  * vcs -version control system
  * build - for maven to build
  * archive & junit results - to  archive artifacts & publish junit results
  * sonar analysis - static code analysis
  * copy the build to any storage - if required
``` js
  pipeline {
    agent { label 'UBUNTU_NODE2'}
   triggers { pollSCM('* * * * *')}
    stages {
        stage('vcs'){
            steps {
                git url: 'https://github.com/srikanthvelma-jenkins/spring-petclinic.git',
                    branch: 'develop'
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
    }
}
```  
  ![preview](images/spcpr5.png)
* As pollSCM is enabled in here it will automatically trigger the pipeline, when the developer pushes the changes to develop branch
* This is the normal scenario for Day Builds 
* But as the developer directly making the commits to develop branch ,it will merge the commits to original code, If developer makes any faulty mistake in his code, it may leads to build failure and it also creates an issue to other developers as it is a failure build
* To overcome this , we may use Pull Request Based work flow
### 2. Create a Jenkins Job for merging pull requset into develop branch
* **Work flow**
* In this type of work flow , normally developers fork the code/repo (develop branch) to his GitHub account, and he writes his code 
* When he wants push/merge the code/changes to main develop branch, now instead of push, he creates a pull request to main develop branch (it may not push/merge authority)
* here we as devops engineers need config/create a pipeline in the main develop branch, such that it will trigger the pipeline on the basis of pull request and it should build the package/artifact.
* Next the result of the build status(pass/failure) should be visible in the main develop branch, so that , the authorized person can decide to merge the changes in to main code or he may reject the pull request
* By doing this , we can easily avoid faulty builds
* To config the pull request based trigger follow below steps
  * need to install  plugin and config `GitHub Pull Request Builder`
  * To config GitHub and jenkins -Credentials are req- we require GitHub token and with that token we can create a `GITHUB_TOKEN`
  ![preview](images/spcpr6.png)
  * To config system. Dashboard=>Manage jenkins=>Configure system
  ![preview](images/spcpr7.png)
  ![preview](images/spcpr8.png)
  * Create a freestyle project with following configuration
  * In General => add a project in GitHUb project url `https://github.com/srikanthvelma-jenkins/spring-petclinic/` 
  ![preview](images/spcpr9.png)
  * In sourece code management => Git => repo url, credentials
  ![preview](images/spcpr10.png)
  * In sourece code management => Git => Advanced => refspec `+refs/pull/*:refs/remotes/origin/pr/*` 
  * In sourece code management =>  Branches to build => branch specifier `${ghprbActualCommit}`
  ![preview](images/spcpr11.png)
  * In Build triggers => GitHub pull request builder => 
  ![preview](images/spcpr12.png)
  ![preview](images/spcpr13.png)
  * In Build Env
  ![preview](images/spcpr14.png)
  * In Build Steps => write build commands
  ![preview](images/spcpr15.png)
  * Next we need to create a webhook in GitHub 
  * Go to repo => settings(project/repo) => webhooks => payload url : `http://<jenkins-ip>:8080/ghprbhook/`, select content type - application/json & let me select individual events - tick pull request & pushes => update hook
  ![preview](images/spcpr16.png) 
  ![preview](images/spcpr17.png)

  * Now let us assume the devloper completes his code and raises a pull request to the main develop branch from his github, it will directly trigger to the jenkins 
  ![preview](images/spcpr19.png)
  ![preview](images/spcpr20.png)
  ![preview](images/spcpr21.png)
  ![preview](images/spcpr22.png)
  ![preview](images/spcpr23.png)
  * As shown above after the build result, we can merge (if any conflict, we have clear that).
  * 
  
  