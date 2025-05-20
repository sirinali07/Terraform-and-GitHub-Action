✅ Goal
Deploy a Docker container (node-hello-app) via GitHub Actions using SSH to a remote Ubuntu EC2 VM.

✅ Prerequisites
* GitHub account
* Basic understanding of Git & GitHub
* Basic knowledge of AWS Cloud


## 🧪 Step-by-Step Lab Guide

### 🔧 Step 1: Prepare  Ubuntu VM (EC2)
 * Launch Ubuntu 22.04 EC2 instance.
 * Allow inbound port 22 (SSH) and 8080 (app) in security group.
 * Create and Download `.pem` file

### 🔧 Step 2: Create a GitHub Repository
* Go to https://github.com.
* Click on "New Repository".
* Name it something like `github-actions-lab`.
* Click Create repository.

### 🧷 Step 3: Add GitHub Secrets
* In your GitHub repo.
* Add the private key (github_ssh_key) to your GitHub repo.
* Go to Settings > 'Secrets and variables' > Actions > New repository secret

| Secret Name | Description                                |
| ----------- | ------------------------------------------ |
| `EC2_HOST`  | Your EC2 public IP or DNS                  |
| `EC2_USER`  | e.g., `ubuntu` or `ec2-user`               |
| `EC2_KEY`   | Private SSH key (entire content of `.pem`) |

### 🗃️ Step 4: Directory Structure of GitHub Repo

![image](https://github.com/user-attachments/assets/ca0e23d0-d3ad-4946-9bd4-b3450cd82ed7)


### 🧷 Step 5: Add `package.json` App Code
```
{
  "name": "example-app",
  "version": "1.0.0",
  "description": "A simple Node.js application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### 🧷 Step 6: Add `index.js` App Code
```
const port = process.env.PORT || 8080;
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  const currentDate = new Date();
  const dateTime = currentDate.toLocaleString();

  res.send(`
    <div style="font-family: Arial, sans-serif; text-align: center; padding: 50px;">
      <h1 style="color: dodgerblue; font-size: 50px;">
        Hello-World!!
      </h1>
      <p style="color: gold; font-size: 20px;">
        Current Date and Time: ${dateTime}
      </p>
    </div>
  `);
});

app.get('/readiness', (req, res) => res.send('Ready !!'));

app.get('/liveness', (req, res) => res.send('Live !!'));

app.listen(port, () => console.log(`Example app listening on port ${port}!`));
```
### 🧷 Step 7: Add `Dockerfile`
```dockerfile
FROM node:18

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 8080
CMD ["npm", "start"]
```

###  🤖  Step 8: Add  `deploy.yml` file in `.github/workflows`

```yaml
name: Build and Deploy to EC2

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Setup Docker
      uses: docker/setup-buildx-action@v3

    - name: Build Docker Image
      run: docker build -t node-hello-app .

    - name: Save Docker Image to Tarball
      run: docker save node-hello-app > node-hello-app.tar

    - name: Copy Docker Image to EC2
      uses: appleboy/scp-action@v0.1.6
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_KEY }}
        source: "node-hello-app.tar"
        target: "~/"

    - name: SSH and Load Docker Image, Run Container
      uses: appleboy/ssh-action@v0.1.10
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_KEY }}
        script: |
          echo "🔄 Updating system"
          sudo apt update -y
          sudo apt install -y curl

          echo "🐳 Installing Docker"
          curl -fsSL https://get.docker.com | sudo sh

          echo "🟢 Enabling and starting Docker service"
          sudo systemctl enable docker
          sudo systemctl restart docker

          echo "🔍 Checking Docker version"
          sudo docker --version

          echo "📦 Loading Docker image"
          sudo docker load < node-hello-app.tar
          
          echo "🛑 Removing existing container (if any)"
          sudo docker rm -f node-hello-app || true

          echo "🚀 Running new container"
          sudo docker run -d --name node-hello-app -p 8080:8080 node-hello-app
```
🔁 This deploys code every time you push to main.
Commit and push the workflow.

### 🧪 STEP 9: Test Deployment

* Go to GitHub → Actions and watch the job.
* After success, open:
```
http://<EC2_PUBLIC_IP>:8080
```
You should see the Node.js Hello App response.
