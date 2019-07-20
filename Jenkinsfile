pipeline{
    agent none
    stages{
        stage("Clone Repository"){
            agent { label 'master' }
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
            agent { label ' master' }
            steps{
                sh "docker build -t mauricio/blog ."
                sh "docker save -o blog.tar mauricio/blog"
                stash name: "stash-artifact", includes: "blog.tar"
                archiveArtifacts 'target/*jar'
            }
        }
        stage("Archive artifact"){
            agent {label 'master'}
            steps{
                archiveArtifacts 'target/*jar'
            }
        }
        stage("Deployment QA"){
            agent { label 'master' }
            steps{
                sh "docker rm blog -f || true"
                sh "docker run -d -p 8090:8090 --name blog mauricio/blog"
            }
        }
        stage("Get Automation Repository"){
            agent { label "master"}
            steps {
               git 'https://github.com/mgviscarra/spring-boot-automation.git' 
            }
            
        }
        stage("Run Automation tests"){
            agent { label "master"}
            steps {
                sh "docker rm browser -f || true"
                sh "docker run -d -p 4444:4444 --name browser --link blog selenium/standalone-chrome"
                sh "mvn test"
            }
            
        }
        stage("Generate Automation report"){
            agent{label "master"}
            steps{
                cucumber buildStatus: 'UNSTABLE',
                fileIncludePattern: 'target/*.json',
                trendsLimit: 10,
                classifications: [
                    [
                        'key': 'Browser',
                        'value': 'Chrome'
                    ]
                ]
            }
        }
        stage("Deployment PROD"){
            agent{label 'PROD'}
            steps{
                 unstash "stash-artifact"
                 sh "docker load -i blog.tar"
                 sh "docker rm blog -f || true"
                 sh "docker run -d -p 8090:8090 --name blog mauricio/blog"
            }
        }
    }
}
