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
        stage("Artifactory"){
            // steps{
            //     script {
                    
            //         def pom = readMavenPom file: 'pom.xml'
            //         def app = docker.build("jhonpridedev/${pom.artifactId}:${pom.version}")

            //         docker.withRegistry('https://registry.hub.docker.com/', 'dockerhub-credentials') {
            //             app.push()
            //             app.push('latest')
            //         }                                        
            //     }            
            // }
        }
        stage("Deploy"){
            // steps{
            //     script {       

            //         def pom = readMavenPom file: 'pom.xml'
            //         sh "docker rm -f ${pom.artifactId}";
            //         sh "docker run -d --name ${pom.artifactId} -p 9966:9966 jhonpridedev/${pom.artifactId}:${pom.version}"

            //     }
            // }
        }
    }
    post {
        success {
            archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
            deleteDir()
        }
    }
}
