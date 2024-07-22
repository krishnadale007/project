# Download CSS-TEMPLATE files(index.html,CSS,images,JS)
```shell
wget https://www.free-css.com/assets/files/free-css-templates/download/page296/healet.zip
```
##upload all CSS-Template files to github Repository
#Create Dockerfile
```shell
FROM nginx:latest
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#Create Jenkins Server(java-17-openjdk)
```shell
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install Jenkins
##INSTALL JAVA PACK
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
alternatives --config java //update java version to 17
openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
$ sudo systemctl enable Jenkins
$ sudo systemctl start Jenkins
$ sudo systemctl status Jenkins
##Now hit PulicIP:8080 on Browser

