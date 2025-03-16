# Jenkins Server Setup in Linux VM #

## Step - 1 : Create Ubuntu VM ##

1) Create Ubuntu VM using AWS EC2 (t2.medium) <br/>
2) Enable 8080 Port Number in Security Group Inbound Rules
3) Connect to VM using MobaXterm

## Step-2 : Instal Java ##

```
sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
java -version
```

## Step-3 : Install Jenkins ##
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```

## Step-4 : Start Jenkins ## 

```
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

## Step-5 : Verify Jenkins ##

```
sudo systemctl status jenkins
```
	
## Step-6 : Open jenkins server in browser using VM public ip ##

```
http://public-ip:8080/
```

## Step-7 : Copy jenkins admin pwd ##
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
	   
## Step-8 : Create Admin Account & Install Required Plugins in Jenkins ##

--------------------------------------------------------------------------------

# **Jenkins Server Setup in Linux VMware Workstation**

## **Step - 1: Create CentOS VM**
1. Create a CentOS VM in VMware Workstation (VM should be CentOS(server with GUI))
2. Enable Port **8080** in the Firewall for External Access
3. Use the Inbuilt Browser in VM or Access from Host Machine


## **Step - 2: Install Java**
```
sudo dnf update -y
sudo yum install fontconfig java-17-openjdk -y
java -version
```

## **Step - 3: Install Jenkins**
```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade -y
sudo yum install jenkins -y
```


## **Step - 4: Start and Enable Jenkins**
```
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```


## **Step - 5: Configure Firewall (For External Access)**
```
sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
sudo firewall-cmd --reload
```


## **Step - 6: Access Jenkins**
- **Inside VM:** Open a browser and visit → `http://localhost:8080`

To get the **initial admin password**, run:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
``` 
