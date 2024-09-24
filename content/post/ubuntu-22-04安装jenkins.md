---
title: ubuntu 22.04安装jenkins
date: 2024-09-24T08:57:00+08:00
---
### Installation of Java
```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
```
### Long Term Support release
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
### Start Jenkins
```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```
