# PROJECT

## **Phase 1: Initial Kubernetes cluster Setup and Deployment**

### 1. Launch EC2 (Ubuntu 22.04): 
- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.

### 2. Install Kubernetes cluster components [On Master & Worker Node]

- Use Script.sh script provided in the repo

### 3. Initialize Kubernetes Master Node [On MasterNode]

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### 4. Join the kubernetes cluster [On WorkerNodes]

```bash
sudo kubeadm join 172.31.20.145:6443 --token mhi5wy.3385kwfnaytaa7mf \
        --discovery-token-ca-cert-hash sha256:ffce702b1a02ff38f47503e83b74d7ad188f49daa31bee1a98e3813b73234a87
```

### 5. Configure Kubernetes Cluster [On MasterNode]

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 6. Deploy Networking Solution (Calico) [On MasterNode]

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 7. Deploy Ingress Controller (NGINX) [On MasterNode]

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```

### 8. Use Kubeaudit to audit the cluster for security information [On MasterNode]

```bash
go to https://github.com/Shopify/kubeaudit/releases
Download the version for linux
wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.2/kubeaudit_0.22.2_linux_amd64.tar.gz
tar -xvzf kubeaudit_0.22.2_linux_amd64.tar.gz
sudo mv kubeaudit /usr/local/bin/
kubeaudit all
```
## **Phase 2: Launch three more servers for Jenkins, Nexus and Sonarqube**

### 1. Launch EC2 (Ubuntu 22.04): 
- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.

## **Phase 3: Sonarqube server setup**

### 1. Install docker: 
```bash 
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
docker ps 
```

### 2. Configure sonarqube server: 
```bash 
Create a project with the name and key BoardGame as we will mention this is jenkins pipeline
Go to administration -> configuration -> webhook -> name -> url http://<JenkinsPublicIP>:8080/sonarqube-webhook/
Create the webhook
Sonarqube webhook settings are very crucial for the sonarqube server to send the quality gate data back to the Jenkins server
```

### 3. Run a docker container with sonarqube image:
```bash 
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

## **Phase 4: Nexus server setup**

### 1. Install docker: 
```bash 
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
docker ps 
```
### 2. Run a docker container with nexus image:
```bash 
docker run -d --name nexus -p 8081:8081 sonatype/nexus3
docker exec -it <nexuscontainerid> /bin/bash
cat /nexus-data/admin.password
```

### 3. Retrieve nexus initial password:
```bash 
docker exec -it <nexuscontainerid> /bin/bash
cat /nexus-data/admin.password
```

### 4. Copy the maven release and maven snapshot URL to add to pom.xml:
```bash 
http://35.176.111.10:8081/repository/maven-snapshots/
http://35.176.111.10:8081/repository/maven-snapshots/
<distributionManagement>
        <repository>
            <id>maven-releases</id>
            <url>http://35.176.111.10:8081/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <url>http://35.176.111.10:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
```

## **Phase 5: Jenkins server setup**

### 1. Install Jenkins: 
```bash 
sudo apt update -y
sudo apt install fontconfig openjdk-17-jre -y
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 2. Install docker: 
```bash 
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
docker ps 
```

### 3. Install the following Plugins on jenkins server: 
```bash 
1. Eclipse Temurin installerVersion - helps with multiple versions of jdk installations
2. Config File Provider - helps to create settings.xml file
3. Pipeline Maven Integration - needed during nexus stage
4. SonarQube Scanner - analyse, generate report and send it to sonarqube server
5. Docker 
6. Docker Pipeline
7. docker-build-step
8. Kubernetes
9. Kubernetes Client API
10. Kubernetes Credentials
11. Kubernetes CLI
```

### 4. Configure the plugins and Create the pipeline: 
```bash 
jenkins -> manage jenkins -> system (Configure servers)
jenkins -> manage jenkins -> tools (Configure installations)
New job -> pipeline job
```

### 5. Install trivy on the server: 
```bash 
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy
```

### 6. Install kubectl on Jenkins server
```bash 
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

## **Phase 6: Initial Kubernetes cluster Setup and Deployment**
### 1. Follow the eks-steps.md file inside the kubernetes directory in this repo. This is to configure role based access controls.
### 2. Once the service account, role and rolebindings have been created and applied for the jenkins user
### 3. Create a service token for jenkins service account 
```bash
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  namespace: webapps
  annotations:
    kubernetes.io/service-account.name: jenkins
```
### 4. Take the token and create a credentials in jenkins as kind secret text.
