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
# sudo git clone https://github.com/kabirbaidhya/react-todo-app /opt/checkout/react-todo-add #This repo will be cloned to the created directory
```
- Build the project
```bash
$ cd /opt/checkout/react-todo-add
$ npm install
$ npm run build
```
- Moving the generated build file to "/opt/deployment/react"
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
```bash
$ sudo apt install nginx
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo nano /etc/nginx/sites-available/react-todo-app

# Add the following configuration
server {
    listen 80;
    server_name your_domain_or_ip;

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




