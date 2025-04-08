Refer this Website for installation: https://www.rosehosting.com/blog/how-to-install-tomcat-on-ubuntu-22-04/
Official Tomcat Download link: https://tomcat.apache.org/download-11.cgi (select the tar file)
1.	Connect your EC2 instance through ssh 
2.	Update all the packages: #sudo apt update
3.	Install java higher than version 8: #sudo apt install openjdk-11-jdk -y
4.	Install Apache tomcat: sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.98/bin/apache-tomcat-9.0.98.tar.gz  (here we will fetch version 9 because version 11 is having some issue) (the version gets updated so check the official link of the tomcat download page )
5.	Unzip the tar file: #sudo tar -xzf file name
Optional step: rename the unzipped file #sudo mv unzipped-file-name new-file-name (eg: tomcat9)
6.	Change the permissions of the file: #sudo chmod -R 777 tomcat9
7.	Go to directory: #cd tomcat9
8.	Go to bin file: #cd bin
9.	To start the tomcat service: #sudo ./startup.sh
To check if tomcat service is started: copy your public ip from instance and paste in into new tab http://<your-public-ip>:8080--------it should show the default tomcat page
By default, Apache Tomcat service runs on 8080 port so edit the security group and add custom tcp but Jenkins also run on port 8080.

For this issue Tomcat has a feature to change the connector port, for that follow steps:
