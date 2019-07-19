pipeline{
    agent none
    stages{
        stage("Clone Repository"){
            agent any
            steps{
                git 'https://github.com/mgviscarra/spring-boot-blog.git'
                sh "echo Cloned!"
            }
            
        }
        stage("Build"){
            agent{
                docker 'maven:3-alpine'
            }
            steps{
                sh "mvn -q clean package"
            }
        }
        stage("Package"){
            agent any
            steps{
                sh "docker build -t mauricio/blog ."
            }
        }
        stage("Deployment QA"){
            agent any
            steps{
                sh "docker run -d -p 8090:8080 --name blog mauricio/blog"
            }
        }
    }
}