pipeline {
    agent any
    stages {
        stage('Build') {         
            agent {
                docker {
                    image 'maven:3.8.8-eclipse-temurin-17-alpine'
                }
            }                 
            steps {
                sh 'mvn clean package -B -ntp -DskipTests'
            }
        }
        // stage('Testing') {
        //     steps {
        //         sh 'mvn test -B -ntp'
        //     }
        //     post {                
        //         success {
        //             jacoco()
        //             junit 'target/surefire-reports/*.xml'
        //         }                
        //         failure {
        //             echo 'Ha ocurrido un error TEST'
        //         }
        //     }
        // }
        // stage('Sonarqube') {
        //     steps {
        //         withSonarQubeEnv('sonarqube'){                    
        //             sh 'mvn sonar:sonar -B -ntp'
        //         }              
        //     }
        // }
        // stage('Quality Gate') {
        //     steps {
        //         // Quality Gate
        //         timeout(time: 1, unit: 'HOURS'){
        //             waitForQualityGate abortPipeline: true
        //         }      
        //     }
        // }        
        // stage('Artifactory') {
        //     steps {
        //         script {
        //             sh 'env | sort'

        //             def pom = readMavenPom file: 'pom.xml'
        //             println pom

        //             def server = Artifactory.server 'artifactory'
        //             def repository = pom.artifactId

        //             if("${GIT_BRANCH}" == 'origin/master'){
        //                 repository = repository + '-release'
        //             } else {
        //                 repository = repository + '-snapshot'
        //             }

        //             def uploadSpec = """
        //                 {
        //                     "files": [
        //                         {
        //                             "pattern": "target/.*.war",
        //                             "target": "${repository}/${pom.groupId}/${pom.artifactId}/${pom.version}/",
        //                             "regexp": "true"
        //                         }
        //                     ]
        //                 }
        //             """
        //             server.upload spec: uploadSpec
        //         }
        //     }
        // }
        stage('Deploy with Ansible') {
            agent {
                docker {
                    image 'quay.io/ansible/ansible-runner:stable-2.12-latest'
                    args '-u root'
                }
            }
            environment {
                ANSIBLE_HOST_KEY_CHECKING = "False"
            }
            options { skipDefaultCheckout() }
            steps {
                dir('ansible'){
                    sshagent (credentials: ['debian-private-key']){
                        sh 'env | sort'

                        // ---  community.general.jboss
                        sh 'pip install --upgrade ansible'
                        sh 'ansible --version'
                        sh 'ansible-galaxy --version'
                        sh 'ansible-galaxy collection install community.general'
                        // ---
                       
                        sh 'ansible-playbook -i hosts deploy_jboss.yml'
                    }
                }
            }
        }
    }
    post {
        success {
            archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
            deleteDir()
        }
    }
}
