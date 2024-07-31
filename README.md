# Integrated DevOps Tools & Created Pipeline 
### GitHub-Docker-Kubernetes-Jenkins
## Download CSS-TEMPLATE Files
###### (index.html,CSS,images,JS)
```shell
curl -O https://www.free-css.com/assets/files/free-css-templates/download/page296/healet.zip
```
#### *Upload all above CSS-Template files to github Repository

## Create Dockerfile & upload to same Repository
```shell
vim Dockerfile
```
```shell
FROM nginx:latest
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Create Jenkins Server(java-17-openjdk):AWS_INSTANCE 
###### Use Ubuntu Image(AMI)
### Connect jenkins(Instance) server & use its Terminal
###### INSTALL JAVA PACK
```shell
sudo apt update
sudo apt install fontconfig openjdk-17-jre
sudo java -version
sudo update-alternatives --config java   #update java version to openjdk version "17.0.8" 2023-07-18
sudo java -version
# openjdk version "17.0.8" 2023-07-18
# OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
# OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
```
###### INSTALL Jenkins
```shell
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
```
sudo apt-get install jenkins -y
```
```shell
sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
```
sudo systemctl status jenkins
```
###### initialAdminPassword
```shell
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
## Now, Hit PublicIP(jenkin_Instance):8080 on Browser to start Jenkins-server
###### PASSWORD >>> cat /var/lib/jenkins/secrets/initialAdminPassword
###  Install suggested plugins & Add Plugins
##### 1.Git
##### 2.Docker
##### 3.Kubernetes
##### 4.Stage-view
##### 5.Aws-Credential
### Add Credentials
##### 1.Docker-creds(USERNAME & TOKEN/PASSWORD)
##### 2.AWS-creds(ACCESS_KEY & SECRETE_KEY)

## Use jenkins_Instance Terminal
###### Install required packages:
```shell
sudo yum install docker -y
sudo yum install git -y
sudo systemctl docker start
sudo systemctl start docker
sudo systemctl enable docker
```
```shell
sudo chmod 666 /var/run/docker.sock
sudo gpasswd -aG jenkins docker
sudo gpasswd -a jenkins docker
sudo chown jenkins /var/run/docker.sock
```
```shell
sudo -i
ll /var/run/docker.sock
```
## Set-Up Kubernetes on Jenkins server
### Create cluster and node(EKS Service)
#### Paste below commands on jenkins server to configure kubernetes
```shell
aws configure
```
```shell
aws eks update-kubeconfig --region ap-south-1 --name my-cluster --kubeconfig /tmp/config        //  ap-south-1(Region_of_Cluster) & my-cluster(cluster_Name)
chown jenkins:jenkins /tmp/config
```
### Install kubectl commands on jenkins server
```shell
sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
sudo chmod +x kubectl
      mkdir -p ~/.local/bin
      mv ./kubectl ~/.local/bin/kubectl
      # and then append (or prepend) ~/.local/bin to $PATH
```
```shell
sudo kubectl cluster-info
```
## Create Manifest-file/yaml-file & upload to Github repository
```shell
vim k8s-pipeline.yml
```
```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: css-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      name: pipeline-tag
  template:
    metadata:
      name: my-css
      labels:
        name: pipeline-tag
    spec: 
      containers:
      - name: docker-jenkins
        image: sohampatil08/devops-tool-jenkins-pipeline      #DockerHub_Image
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: css-service
spec:
  selector:
    name: pipeline-tag      #This tag should match the labels in the deployment
  ports:
  - name: http
    targetPort: 80
    port: 80
    protocol: TCP
  type: LoadBalancer
```
## Create Job on jenkins server (GUI)
### Create item>Pipeline>SCM_Git_Repository:Github_file:<jenkins-pipeline(jenkinsfile)>
#
## Create jenkins-pipeline(jenkinsfile) & upload to Github repository
```shell
vim devops-tool-pipeline
```
```shell
pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = '<sohampatil08/devops-tool-jenkins-pipeline>'     #Mention your Dockerhub_Repository
    }

    stages {
        stage('Pull Source Code') {
            steps {
                git 'https://github.com/soham08022001/DevopsTool-Jenkins-Integration.git'    #Mention your Github_Repository
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKERHUB_REPO}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            environment {
                registryCredential = 'docker-creds'    #Mention docker-credential NAME here which you've created in JENKINS-credentials
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', registryCredential) {
                        sh 'docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            environment {
                AWS_CREDENTIALS = 'aws-creds'      #Mention AWS-credential NAME here which you've created in JENKINS-credentials
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS]]) {
                    script {
                        sh """
                    aws eks update-kubeconfig --name my-cluster --region ap-south-1 --kubeconfig /tmp/config
                    kubectl apply -f k8s-pipeline.yml  --kubeconfig=/tmp/config
                    kubectl set image deployment/css-deployment docker-jenkins=sohampatil08/devops-tool-jenkins-pipeline:${env.BUILD_NUMBER}  --kubeconfig=/tmp/config
                    """
                    }
                }
            }
        }
    }
}
```
### Last stage of pipeline explained here:
#### stage('Deploy to Kubernetes') > script {sh""" -//- """}
######  Mention your Cluster_Name at my-cluster & Region at ap-south-1
#### aws eks update-kubeconfig --name my-cluster --region ap-south-1 --kubeconfig /tmp/config
#
###### Mention your YAML_file name at k8s-pipeline.yml
#### kubectl apply -f k8s-pipeline.yml  --kubeconfig=/tmp/config
#
###### kubectl set image deployment/<Deployment_Name> <Container_Name>=<DockerHub_Repository>:${env.BUILD_NUMBER}  --kubeconfig=/tmp/config
#### kubectl set image deployment/css-deployment docker-jenkins=sohampatil08/devops-tool-jenkins-pipeline:${env.BUILD_NUMBER}  --kubeconfig=/tmp/config
This command updates the docker-jenkins container image in the <css-deployment> deployment to a specific version of the <sohampatil08/devops-tool-jenkins-pipeline> image, determined by the build number from a CI/CD pipeline, using a specific kubeconfig file to connect to the cluster.
#
### After successfully uploaded all the files on GitHub Repository, On jenkins-server(GUI) hit the > Build
