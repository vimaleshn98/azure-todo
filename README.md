# Sample To Do List web application using Spring Boot and MySQL

A simple Todo list application using Spring Boot with the following options:

- Spring JPA and MySQL for data persistence
- Thymeleaf templae for the rendering.

To build and run the sample from a fresh clone of this repo:

## Configure MySQL

1. Create a database in your MySQL instance.
2. Update the application.properties file in the `src/main/resources` folder with the URL, username and password for your MySQL instance. The table schema for the Todo objects will be created for you in the database.


## Build and run the sample

1. `mvnw package`
3. `java -jar target/TodoDemo-0.0.1-SNAPSHOT.jar`
3. Open a web browser to http://localhost:8080

As you add and update tasks in the app you can verify the changes in the database through the MySQL console using simple statements like 
`select * from todo_item`.




1 build :

stage("build"){
            steps{
                sh 'mvn clean'
                sh 'mvn -f app/pom.xml compile'
            }
            post{
                success{
                    echo "========Maven compile stage executed successfully========"
                }
                failure{
                    echo "========Maven compile stage execution failed========"
                }
            }
            
  pom file inside app 
  
 2 test stores data .xml
  stage("Test"){
            steps{
                echo "Maven Test"
                sh 'mvn -f app/pom.xml -Dmaven.test.failure.ignore=true  test'
                
            }
            post{
                success{
                    echo "========Maven Test stage executed successfully========"
                     junit 'app/target/surefire-reports/*.xml'

                }
                failure{
                    echo "========Maven Test stage execution failed========"
                     junit 'app/target/surefire-reports/*.xml'
                }
            }
        }
        
       3 sonar analysis and package maven:
       
       stage('sonar analysis'){
            agent any
            steps{
                withSonarQubeEnv('sonarcloud'){
                    echo 'Performing SonarQube Analysis'
                    sh 'mvn -f app/pom.xml package sonar:sonar'
                }
            }
            post{
                success{
                    echo "========Code Analysis stage executed successfully========"

                }
                failure{
                    echo "========Code Analysis stage execution failed========"
                }
            }
            
        }
  ![image](https://user-images.githubusercontent.com/52690867/114670001-2cbc8a00-9d20-11eb-9680-0c6f81110aae.png)
        confiuration on jenkins
        
        sonar cloud :
        create new organzation and project.
        get token and add that to pom.xml
        
        
        4 .artifacte creation:
          jforg create one repo their.
          https://my.jfrog.com/login
 ![image](https://user-images.githubusercontent.com/52690867/114670396-90df4e00-9d20-11eb-87c0-1bbdcf0d9fe9.png)
           stage("Deployee"){
           when {
                expression {
                        currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
                steps{
     
                     rtUpload (
                         serverId: 'artifactory-server',
                     spec: """{
                             "files": [
                                      {
                                     "pattern": "/var/lib/jenkins/workspace/TodoApp@2/app/target/*.war",
                                     "target": "art-doc-dev-locs/"
                                    }
                                ]
                            }"""
                        )
                    }
            post{
                success{
                    echo "========Deploying executed successfully========"

                }
                
                failure{
                    echo "========Deploying stage execution failed========"
                }
            }
        }    
        stage("Download"){
           when {
                expression {
                        currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
                steps{
                     
            rtDownload (
                         serverId: 'artifactory-server',
                     spec: """{
                             "files": [
                                      {
                                      "pattern": "art-doc-dev-locs/*.war",
                                      "target": "app/"
                                    }
                                ]
                            }"""
                        )
                    }
            post{
                success{
                    echo "========Download executed successfully ========"
                    // sshagent(['ubuntu2']){
                    // sh 'scp -r bazinga/*.jar ubuntu@18.236.173.67:/home/ubuntu/artifacts'
                    }
                
                failure{
                    echo "========Download stage execution failed========"
                }
            }
        }
        
        
        6 docker:
            stage('Docker build'){
            steps{
               
                    sh 'docker image prune -a --force'
                    sh 'docker-compose build'
                    sh 'docker ps'
                
                
            }
        }
        stage('Pushing images to docker hub'){
            steps{
                

                withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerHubPWD')]) {
                            // some block
                   sh "docker login -u 7349216534 -p ${dockerHubPWD}"

                }
                sh "docker image tag todocicd_app:latest 7349216534/todo-app:v${env.BUILD_ID}"
                sh "docker push 7349216534/todo-app:v${env.BUILD_ID}"

            }
        }
        
        
        add credentail like dockerhub (easy goto syntax then search withcreandential and add their)
        
        
        
        7: kube config:
        
        stage('deploying it to kubernetes'){
            steps{
                sh 'chmod +x change-tag.sh'
                sh """./change-tag.sh v${env.BUILD_ID}"""
                sh 'cat k8s/api-deployment.yaml'
                withKubeConfig(caCertificate: '', clusterName: 'end-to-end-k8s', contextName: 'end-to-end-k8s', credentialsId: 'kube-azure', namespace: '', serverUrl: 'end-to-end-k8s-dns-db7293cf.hcp.westus2.azmk8s.io'){
                                // some block
                    sh 'kubectl apply -f k8s/database-deployment.yaml'
                    
                }
                sleep(120)
               withKubeConfig(caCertificate: '', clusterName: 'end-to-end-k8s', contextName: 'end-to-end-k8s', credentialsId: 'kube-azure', namespace: '', serverUrl: 'end-to-end-k8s-dns-db7293cf.hcp.westus2.azmk8s.io') {
                                // some block
                    sh 'kubectl apply -f k8s/api-deployment.yaml'
                    sh 'kubectl get pods'
                    sh 'kubectl get svc'
                    
                }

            }
        }
        
        
        add kube config file on creadential and try use some more cmd on that.
  in jforg:
![image](https://user-images.githubusercontent.com/52690867/114672800-2845a080-9d23-11eb-8a3f-5b04f2150884.png)
Goto Repository -> repository -> create new repo -> selectet maven -> given proper naming convention.
create


azure cluster creation:
az aks get-credentials --resource-group end-to-end --name end-to-end-k8s

![image](https://user-images.githubusercontent.com/52690867/115217963-ad65f680-a123-11eb-8549-805b743f9989.png)

