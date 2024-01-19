# React-App-Deployment
<p>In the React Todo List Project Deployment, we established a robust DevOps framework encompassing user setup, React project deployment, Nginx proxy configuration, CICD with Jenkins, custom shell scripting for Java installation, and Dockerization. We initiated by creating a privileged "DevOps" user in Ubuntu, then cloned, built, and deployed the React project using NodeJS, npm, and pm2. Nginx was configured to proxy requests, and firewall rules was enforced. CICD was streamlined through Jenkins, facilitating code pull, build, deployment, and S3 uploads. Custom bash scripting ensured Java setup and logging, while Dockerization encapsulated the React app, managed through Docker-compose. This end-to-end approach optimizes project development, deployment, and maintenance.
</p>

## Instructions
<p>All work to be done on Ec2 instance</p>

Part 1 - Create a sudo user in Ubuntu
<p>Login to Linux create a user and add that user to the sudo group</p>

```bash
$ sudo adduser react
$ sudo usermod -aG sudo DevOps
$ su - react #To shift to react user
```

Part 2 - Checkout, Build, and Deploy the app
 - Install nodejs and npm
```bash
$ sudo apt update
$ sudo apt install nodejs npm
```
- Clone the repo
```bash
$ sudo mkdir -p /opt/checkout
$ sudo git clone https://github.com/kabirbaidhya/react-todo-app /opt/checkout/react-todo-add #This repo will be cloned to the created directory
```
- Build the project
```bash
$ cd /opt/checkout/react-todo-add
$ npm install
$ npm run build
```
- Moving the generated build file to "/opt/deployment/react"
<p>Build to be done in parent directory "/opt/checkout/react-todo-add"</p>

```bash
$ mkdir -p /opt/deployment/react
$ sudo cp -r build/* /opt/deployment/react
```
- Now deploy the project using pm2
```bash
$ npm install -g pm2 #This will install pm2 globally
$ pm2 start /opt/deployment/react/index.html --name react-todo-app
```

Part 3 - Serve the project

- Setup nginx proxy
<p>Use reverse proxy</p>

```bash
$ sudo apt install nginx
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo nano /etc/nginx/sites-available/react-todo-app

# Add the following configuration
server {
    listen 80;
    server_name your_Instance_Ip;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

$ sudo ln -s /etc/nginx/sites-available/react-todo-app /etc/nginx/sites-enabled/
$ sudo systemctl restart nginx
```

- Configure UFW
```bash
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 22/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 443/tcp
$ sudo ufw allow 3000/tcp
$ sudo ufw enable
```

Part 4 - CI/CD Pipeline

- Install jnekins
```bash
$ sudo apt install openjdk-8-jdk
$ wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
$ sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo apt update
$ sudo apt install jenkins
```
<p>Access Jenkins at http://<your-instance-ip>:8080, get the admin password
1. Create Jenkins pipeline job
2. Install S3 plugin
3. Update pipeline and upload the build using the s3 credential </p>

Part 5 - Shell Scripting 

- Create a bash script to download jdk and assign a crontab
```bash
$ sudo nano /opt/scripts/install_java.sh

# Add the following content
#!/bin/bash
echo "Downloading and installing Open JDK 1.8"
sudo apt-get install openjdk-8-jdk -y
echo "export PATH=$PATH:/usr/lib/jvm/java-8-openjdk-amd64/bin" >> ~/.bashrc
source ~/.bashrc
```
- Make the change to permission to the script
```bash
$ sudo chmod +x /opt/scripts/install_java.sh
```

- Schedule the script to run at startup:
<P>Crontab is used to schedule job when to run</P>

```bash
$ (sudo crontab -l ; echo "@reboot /opt/scripts/install_java.sh >> /opt/logs/script_logs.log 2>&1") | sudo crontab -
```

Part 6 - Dockerize the React app
<p>Install docker and docker-compose before these commands</p>

```bash
$ cd /opt/checkout/react-todo-add
$ sudo nano Dockerfile

# Add the following content
FROM node:14-alpine
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

- Create a docker-compose.yml file:-
```bash
version: '3'
services:
  react-app:
    build:
      context: .
    ports:
      - "3000:3000"
```
- Run the container
<p>Run the container and expose on the port 3000</p>

```bash
$ sudo docker-compose build
$ sudo docker-compose up -d
```
<p>Do change the ec2 instance security group and add new rule for port 3000
- Run on HTTP://your_instance_ip:3000 </p>





