pipeline {
    agent {
        docker {
            image 'maven:3.8.8-eclipse-temurin-17-alpine'
        }
    } 
    stages {
        stage('Build') {                       
            steps {
                sh 'mvn clean package -B -ntp -DskipTests'
            }
        }
        stage('Testing') {
            steps {
                sh 'mvn test -B -ntp'
            }
            post {                
                success {
                    jacoco()
                    junit 'target/surefire-reports/*.xml'
                }                
                failure {
                    echo 'Ha ocurrido un error TEST'
                }
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonarqube'){                    
                    sh 'mvn sonar:sonar -B -ntp'
                }              
            }
        }
        stage('Quality Gate') {
            steps {
                // Quality Gate
                timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: true
                }      
            }
        }        
        stage('Artifactory') {
            steps {
                script {
                    sh 'env | sort'

                    def pom = readMavenPom file: 'pom.xml'
                    println pom

                    def server = Artifactory.server 'artifactory'
                    def repository = 'spring-framework-petclinic'

                    if("${GIT_BRANCH}" == 'origin/master'){
                        repository = repository + '-release'
                    } else {
                        repository = repository + '-snapshot'
                    }

                    def uploadSpec = """
                        {
                            "files": [
                                {
                                    "pattern": "target/.*.war",
                                    "target": "${repository}/${pom.groupId}/${pom.artifactId}/${pom.version}/",
                                    "regexp": "true"
                                }
                            ]
                        }
                    """
                    server.upload spec: uploadSpec
                }
            }
        }
        // stage("Deploy"){
            // steps{
            //     script {       

            //         def pom = readMavenPom file: 'pom.xml'
            //         sh "docker rm -f ${pom.artifactId}";
            //         sh "docker run -d --name ${pom.artifactId} -p 9966:9966 jhonpridedev/${pom.artifactId}:${pom.version}"

            //     }
            // }
        // }
    }
    post {
        success {
            archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
            deleteDir()
        }
    }
}
