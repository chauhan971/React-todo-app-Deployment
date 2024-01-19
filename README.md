# React-App-Deployment
<p>In the React Todo List Project Deployment, we established a robust DevOps framework encompassing user setup, React project deployment, Nginx proxy configuration, CICD with Jenkins, custom shell scripting for Java installation, and Dockerization. We initiated by creating a privileged "DevOps" user in Ubuntu, then cloned, built, and deployed the React project using NodeJS, npm, and pm2. Nginx was configured to proxy requests, and firewall rules was enforced. CICD was streamlined through Jenkins, facilitating code pull, build, deployment, and S3 uploads. Custom bash scripting ensured Java setup and logging, while Dockerization encapsulated the React app, managed through Docker-compose. This end-to-end approach optimizes project development, deployment, and maintenance.
</p>

## Instructions
<p>All work to be done on Ec2 instance</p>

Part 1 - Create a sudo user in Ubuntu
<p>Login to Linux create a user and add that user to the sudo group</p>
```bash
$ sudo adduser react
$ sudo usermod -aG sudo devops
```
