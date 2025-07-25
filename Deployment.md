 														DevOps Project

 								**Deployment of Microservices Application using Ingress Controller**

 														   *by*

 										 			**Kastro Kiran V**





\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

**Step 1: Basic Setup**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Push the code from Local to Remote



Use a Personal Access Token (HTTPS method)

---

🔧 Step-by-Step:

Go to GitHub → Developer Settings > Personal Access Tokens

Click "Tokens (classic)" > Generate new token

Set scopes (permissions):

Select at least:

 	repo (for full control of private repositories)

 	workflow (if you're using GitHub Actions)

 	Copy the token (you won't be able to see it again!)



🔁 Update Git credentials:

git remote set-url origin https://<your\_username>:<your\_token>@github.com/KastroVKiran/microservices-ingress-kastro.git



✅ Push the code

git push -u origin master

================================================================

1.1. Launch 1 VM (Ubuntu, 24.04, t2.large, 28 GB, Name: Ingress-Server)

================================================================

Open below ports for the Security Group attached to the above VM

Type                  Protocol   Port range

SMTP                  TCP           25

(Used for sending emails between mail servers)



Custom TCP        TCP		3000-10000

(Used by various applications, such as Node.js (3000), Grafana (3000), Jenkins (8080), and custom web applications.



HTTP                   TCP           80

Allows unencrypted web traffic. Used by web servers (e.g., Apache, Nginx) to serve websites over HTTP.



HTTPS                 TCP           443

Allows secure web traffic using SSL/TLS.



SSH                      TCP           22

Secure Shell (SSH) for remote server access.



Custom TCP         TCP           6443

Kubernetes API server port. Used for communication between kubectl, worker nodes, and the Kubernetes control plane.



SMTPS                 TCP           465

Secure Mail Transfer Protocol over SSL/TLS. Used for sending emails securely via SMTP with encryption.



Custom TCP         TCP           30000-32767

Kubernetes NodePort service range.



\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

**Step 2: Tools Installation**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

=============================

2.1.1 Connect to the Ingress Server

=============================

vi Jenkins.sh ----> Paste the below content ---->

\#!/bin/bash



\# Update system

sudo apt update -y



\# Install dependencies

sudo apt install -y fontconfig openjdk-17-jre-headless wget gnupg2



\# Download and add the Jenkins GPG key

wget -O- https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | \\

    gpg --dearmor | sudo tee /usr/share/keyrings/jenkins-keyring.gpg > /dev/null



\# Add Jenkins repository

echo "deb \[signed-by=/usr/share/keyrings/jenkins-keyring.gpg] https://pkg.jenkins.io/debian-stable binary/" | \\

    sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null



\# Update package lists

sudo apt update -y



\# Install Jenkins

sudo apt install jenkins -y



\# Start Jenkins

sudo systemctl start jenkins

sudo systemctl enable jenkins



\# Print status

sudo systemctl status jenkins



----> esc ----> :wq ----> sudo chmod +x jenkins.sh ----> ./jenkins.sh



Open Port 8080 in Jenkins server

Access Jenkins and setup Jenkins



=============================

2.1.2 Install Docker

=============================

vi docker.sh ----> Paste the below content ---->



\#!/bin/bash



\# Update package manager repositories

sudo apt-get update



\# Install necessary dependencies

sudo apt-get install -y ca-certificates curl



\# Create directory for Docker GPG key

sudo install -m 0755 -d /etc/apt/keyrings



\# Download Docker's GPG key

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc



\# Ensure proper permissions for the key

sudo chmod a+r /etc/apt/keyrings/docker.asc



\# Add Docker repository to Apt sources

echo "deb \[arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \\

$(. /etc/os-release \&\& echo "$VERSION\_CODENAME") stable" | \\

sudo tee /etc/apt/sources.list.d/docker.list > /dev/null



\# Update package manager repositories

sudo apt-get update



sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin



 ----> esc ----> :wq ----> sudo chmod +x docker.sh ----> ./docker.sh



docker --version

Try to pull some default image ----> docker pull hello-world ----> If you are unable to pull the image, execute the below command to provide necessary permissions ----> sudo chmod 666 /var/run/docker.sock ----> docker pull hello-world ----> You will be able to pull the image



\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

**Step 3: Access Jenkins Dashboard**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Setup the Jenkins



3.1. Plugins installation

Install below plugins;

Docker, Docker Commons, Docker Pipeline, Docker API, docker-build-step, AWS Credentials, Pipeline stage view, Kubernetes, Kubernetes CLI, Kubernetes Client API, Kubernetes Credentials, Config File Provider, Prometheus metrics



3.2. Creation of Credentials

Configure Dockerhub Credentials as "dockerhub-creds"

Configure AWS Credentials (Access and Secret Access Keys) as "aws-creds"



3.3. Tools Configuration

Manage Jenkins ---> System Configuration ---> Tools ---> Docker Installations ---> Name: docker, 'Check' Install automatically,  Click on 'Add Installer', Select 'Download from docker.com', Version: latest ---> Apply ---> Save



\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

**Step 4: Creation of EKS Cluster**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

4.1. Creation of IAM user (To create EKS Cluster, its not recommended to create using Root Account)



4.2. Attach policies to the user

 AmazonEC2FullAccess, AmazonEKS\_CNI\_Policy, AmazonEKSClusterPolicy, AmazonEKSWorkerNodePolicy, AWSCloudFormationFullAccess, IAMFullAccess



Attach the below inline policy also for the same user

{

  "Version": "2012-10-17",

  "Statement": \[

    {

      "Sid": "VisualEditor0",

      "Effect": "Allow",

      "Action": "eks:\*",

      "Resource": "\*"

    }

  ]

}





4.3. Create Access Keys for the user created



With this we have created the IAM User with appropriate permissions to create the EKS Cluster



4.4. Install AWS CLI (to interact with AWS Account)

sudo apt update



curl "https://awscli.amazonaws.com/awscli-exe-linux-x86\_64.zip" -o "awscliv2.zip"

sudo apt install unzip

unzip awscliv2.zip

sudo ./aws/install



Configure aws by executing below command

aws configure



4.5. Install KubeCTL (to interact with K8S)

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin

kubectl version --short --client



4.6. Install EKS CTL (used to create EKS Cluster)

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl\_$(uname -s)\_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version



4.7. Create EKS Cluster

eksctl create cluster --name kastro-cluster --region us-east-1 --node-type t2.medium --zones us-east-1a,us-east-1b



4.8. Modifying the permissions

sudo usermod -aG docker jenkins

sudo systemctl restart docker

sudo systemctl restart jenkins



4.9. Installation of Ingress Controller

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/aws/deploy.yaml



\#Wait for sometime for the pods to create

kubectl get pods -n ingress-nginx



\#To get the external ip of ingress

kubectl get svc ingress-nginx-controller -n ingress-nginx



4.10. To delete the cluster (Optional)

eksctl delete cluster --name kastro-cluster --region us-east-1



\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

**Step 5: Creation of Jenkins Job**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

pipeline {

    agent any



    environment {

        DOCKER\_HUB\_REPO = 'kastrov/techsolutions-app'

        K8S\_CLUSTER\_NAME = 'kastro-cluster'

        AWS\_REGION = 'us-east-1'

        NAMESPACE = 'default'

        APP\_NAME = 'techsolutions'

    }



    stages {

        stage('Checkout') {

            steps {

                echo 'Checking out source code...'

                git 'https://github.com/KastroVKiran/microservices-ingress-kastro.git'

            }

        }



        stage('Build Docker Image') {

            steps {

                echo 'Building Docker image...'

                script {

                    def buildNumber = env.BUILD\_NUMBER

                    def imageTag = "${DOCKER\_HUB\_REPO}:${buildNumber}"

                    def latestTag = "${DOCKER\_HUB\_REPO}:latest"



                    sh "docker build -t ${imageTag} ."

                    sh "docker tag ${imageTag} ${latestTag}"



                    env.IMAGE\_TAG = buildNumber

                }

            }

        }



        stage('Push to DockerHub') {

            steps {

                echo 'Pushing Docker image to DockerHub...'

                script {

                    withCredentials(\[usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER\_PASSWORD', usernameVariable: 'DOCKER\_USERNAME')]) {

                        sh "echo \\${DOCKER\_PASSWORD} | docker login -u \\${DOCKER\_USERNAME} --password-stdin"

                        sh "docker push ${DOCKER\_HUB\_REPO}:${env.IMAGE\_TAG}"

                        sh "docker push ${DOCKER\_HUB\_REPO}:latest"

                    }

                }

            }

        }



        stage('Configure AWS and Kubectl') {

            steps {

                echo 'Configuring AWS CLI and kubectl...'

                script {

                    withCredentials(\[\[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {

                        sh "aws configure set region ${AWS\_REGION}"

                        sh "aws eks update-kubeconfig --region ${AWS\_REGION} --name ${K8S\_CLUSTER\_NAME}"

                        sh "kubectl config current-context"

                        sh "kubectl get nodes"

                    }

                }

            }

        }



        stage('Deploy to Kubernetes') {

            steps {

                echo 'Deploying application to Kubernetes...'

                script {

                    withCredentials(\[\[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {

                        sh "sed -i 's|kastrov/techsolutions-app:latest|kastrov/techsolutions-app:${env.IMAGE\_TAG}|g' k8s/deployment.yaml"

                        sh "kubectl apply -f k8s/deployment.yaml"

                        sh "kubectl rollout status deployment/${APP\_NAME}-deployment --timeout=300s"

                        sh "kubectl get pods -l app=${APP\_NAME}"

                        sh "kubectl get svc ${APP\_NAME}-service"

                    }

                }

            }

        }



        stage('Deploy Ingress') {

            steps {

                echo 'Deploying Ingress resource...'

                script {

                    withCredentials(\[\[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {

                        sh "kubectl apply -f k8s/ingress.yaml"

                        sleep(10)

                        sh "kubectl get ingress ${APP\_NAME}-ingress"

                        sh "kubectl describe ingress ${APP\_NAME}-ingress"

                    }

                }

            }

        }



        stage('Get Ingress URL') {

            steps {

                echo 'Getting Ingress URL...'

                script {

                    withCredentials(\[\[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {

                        timeout(time: 10, unit: 'MINUTES') {

                            waitUntil {

                                script {

                                    def result = sh(

                                        script: "kubectl get svc ingress-nginx-controller -n ingress-nginx -o jsonpath='{.status.loadBalancer.ingress\[0].hostname}'",

                                        returnStdout: true

                                    ).trim()



                                    if (result \&\& result != '') {

                                        env.INGRESS\_URL = "http://${result}"

                                        echo "Ingress URL: ${env.INGRESS\_URL}"

                                        return true

                                    }

                                    return false

                                }

                            }

                        }



                        echo "========================================="

                        echo "DEPLOYMENT SUCCESSFUL!"

                        echo "========================================="

                        echo "Application URL: ${env.INGRESS\_URL}"

                        echo ""

                        echo "Available Paths:"

                        echo "- Home Page: ${env.INGRESS\_URL}/"

                        echo "- About Page: ${env.INGRESS\_URL}/about"

                        echo "- Services Page: ${env.INGRESS\_URL}/services"

                        echo "- Contact Page: ${env.INGRESS\_URL}/contact"

                        echo "========================================="



                        sh "curl -I ${env.INGRESS\_URL}/ || echo 'Home page check failed'"

                        sh "curl -I ${env.INGRESS\_URL}/about || echo 'About page check failed'"

                        sh "curl -I ${env.INGRESS\_URL}/services || echo 'Services page check failed'"

                        sh "curl -I ${env.INGRESS\_URL}/contact || echo 'Contact page check failed'"

                    }

                }

            }

        }

    }



    post {

        always {

            echo 'Cleaning up Docker images...'

            sh "docker rmi ${DOCKER\_HUB\_REPO}:${env.IMAGE\_TAG} || true"

            sh "docker rmi ${DOCKER\_HUB\_REPO}:latest || true"

        }



        success {

            echo 'Pipeline completed successfully!'

            echo "Access your application at: ${env.INGRESS\_URL}"

        }



        failure {

            echo 'Pipeline failed! Please check the logs.'

        }

    }

}



\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

**Step 6: Monitoring**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Launch Ubuntu VM, 22.04, t2.medium,

Name the VM as Monitoring Server



9.1. Connect to the Monitoring Server VM (Execute in Monitoring Server VM)

Create a dedicated Linux user sometimes called a 'system' account for Prometheus

sudo apt update



sudo useradd \\

    --system \\

    --no-create-home \\

    --shell /bin/false prometheus



With the above command, we have created a 'Prometheus' user



Explanation of above command

–system – Will create a system account.

–no-create-home – We don’t need a home directory for Prometheus or any other system accounts in our case.

–shell /bin/false – It prevents logging in as a Prometheus user.

Prometheus – Will create a Prometheus user and a group with the same name.



9.2. Download the Prometheus

sudo wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz

tar -xvf prometheus-2.47.1.linux-amd64.tar.gz

sudo mkdir -p /data /etc/prometheus

cd prometheus-2.47.1.linux-amd64/



Move the Prometheus binary and a promtool to the /usr/local/bin/. promtool is used to check configuration files and Prometheus rules.

sudo mv prometheus promtool /usr/local/bin/



Move console libraries to the Prometheus configuration directory

sudo mv consoles/ console\_libraries/ /etc/prometheus/



Move the example of the main Prometheus configuration file

sudo mv prometheus.yml /etc/prometheus/prometheus.yml



Set the correct ownership for the /etc/prometheus/ and data directory

sudo chown -R prometheus:prometheus /etc/prometheus/ /data/



Delete the archive and a Prometheus tar.gz file

cd

You are in ~ path

rm -rf prometheus-2.47.1.linux-amd64.tar.gz



prometheus --version

You will see as "version 2.47.1"



prometheus --help



We’re going to use Systemd, which is a system and service manager for Linux operating systems. For that, we need to create a Systemd unit configuration file.

sudo vi /etc/systemd/system/prometheus.service ---> Paste the below content ---->



\[Unit]

Description=Prometheus

Wants=network-online.target

After=network-online.target

StartLimitIntervalSec=500

StartLimitBurst=5

\[Service]

User=prometheus

Group=prometheus

Type=simple

Restart=on-failure

RestartSec=5s

ExecStart=/usr/local/bin/prometheus \\

  --config.file=/etc/prometheus/prometheus.yml \\

  --storage.tsdb.path=/data \\

  --web.console.templates=/etc/prometheus/consoles \\

  --web.console.libraries=/etc/prometheus/console\_libraries \\

  --web.listen-address=0.0.0.0:9090 \\

  --web.enable-lifecycle

\[Install]

WantedBy=multi-user.target



 ----> esc ----> :wq ---->



To automatically start the Prometheus after reboot run the below command

sudo systemctl enable prometheus



Start the Prometheus

sudo systemctl start prometheus



Check the status of Prometheus

sudo systemctl status prometheus



Open Port No. 9090 for Monitoring Server VM and Access Prometheus

[public-ip:9090](public-ip:9090)



If it doesn't work, in the web link of browser, remove 's' in 'https'. Keep only 'http' and now you will be able to see.

You can see the Prometheus console.

Click on 'Status' dropdown ---> Click on 'Targets' ---> You can see 'Prometheus (1/1 up)' ----> It scrapes itself every 15 seconds by default.



10\. Install Node Exporter (Execute in Monitoring Server VM)

You are in ~ path now



Create a system user for Node Exporter and download Node Exporter:

sudo useradd --system --no-create-home --shell /bin/false node\_exporter

wget https://github.com/prometheus/node\_exporter/releases/download/v1.6.1/node\_exporter-1.6.1.linux-amd64.tar.gz



Extract Node Exporter files, move the binary, and clean up:

tar -xvf node\_exporter-1.6.1.linux-amd64.tar.gz

sudo mv node\_exporter-1.6.1.linux-amd64/node\_exporter /usr/local/bin/

rm -rf node\_exporter\*



node\_exporter --version



Create a systemd unit configuration file for Node Exporter:

sudo vi /etc/systemd/system/node\_exporter.service



Add the following content to the node\_exporter.service file:

\[Unit]

Description=Node Exporter

Wants=network-online.target

After=network-online.target



StartLimitIntervalSec=500

StartLimitBurst=5



\[Service]

User=node\_exporter

Group=node\_exporter

Type=simple

Restart=on-failure

RestartSec=5s

ExecStart=/usr/local/bin/node\_exporter --collector.logind



\[Install]

WantedBy=multi-user.target



Note: Replace --collector.logind with any additional flags as needed.



Enable and start Node Exporter:

sudo systemctl enable node\_exporter

sudo systemctl start node\_exporter



Verify the Node Exporter's status:

sudo systemctl status node\_exporter

You can see "active (running)" in green colour

Press control+c to come out of the file



9.3. Configure Prometheus Plugin Integration



As of now we created Prometheus service, but we need to add a job in order to fetch the details by node exporter. So for that we need to create 2 jobs, one with 'node exporter' and the other with 'jenkins' as shown below;



Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.



Prometheus Configuration:



To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file.

The path of prometheus.yml is; cd /etc/prometheus/ ----> ls -l ----> You can see the "prometheus.yml" file ----> sudo vi prometheus.yml ----> You will see the content and also there is a default job called "Prometheus" Paste the below content at the end of the file;



  - job\_name: 'node\_exporter'

    static\_configs:

      - targets: \['<MonitoringVMip>:9100']



  - job\_name: 'jenkins'

    metrics\_path: '/prometheus'

    static\_configs:

      - targets: \['<your-jenkins-ip>:<your-jenkins-port>']



OR PASTE THE BELOW CONTENT DIRECTLY

scrape\_configs:

  # The job name is added as a label `job=<job\\\_name>` to any timeseries scraped from this config.

  - job\_name: "prometheus"



    # metrics\_path defaults to '/metrics'

    # scheme defaults to 'http'.

    static\_configs:

      - targets: \['localhost:9090']



  - job\_name: 'node\_exporter'

    static\_configs:

      - targets: \['<MonitoringVMip>:9100']



  - job\_name: 'jenkins'

    metrics\_path: '/prometheus'

    static\_configs:

      - targets: \['<your-jenkins-ip>:<your-jenkins-port>']





 In the above, replace <your-jenkins-ip> and <your-jenkins-port> with the appropriate IPs ----> esc ----> :wq

Also replace the public ip of monitorting VM. Dont change 9100. Even though the Monitoring server is running on 9090, dont change 9100 in the above script



Check the validity of the configuration file:

promtool check config /etc/prometheus/prometheus.yml

You should see "SUCCESS" when you run the above command, it means every configuration made so far is good.



Reload the Prometheus configuration without restarting:

curl -X POST http://localhost:9090/-/reload



Access Prometheus in browser (if already opened, just reload the page):

http://<your-prometheus-ip>:9090/targets



For Node Exporter you will see (0/1) in red colour. To resolve this, open Port number 9100 for Monitoring VM



You should now see "Jenkins (1/1 up)" "node exporter (1/1 up)" and "prometheus (1/1 up)" in the prometheus browser.

Click on "showmore" next to "jenkins." You will see a link. Open the link in new tab, to see the metrics that are getting scraped



---

10\. Install Grafana (Execute in Monitoring Server VM)

---

You are currently in /etc/Prometheus path.



Install Grafana on Monitoring Server;



Step 1: Install Dependencies:

First, ensure that all necessary dependencies are installed:

sudo apt-get update

sudo apt-get install -y apt-transport-https software-properties-common



Step 2: Add the GPG Key:

cd ---> You are now in ~ path

Add the GPG key for Grafana:

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -



You should see OK when executed the above command.



Step 3: Add Grafana Repository:

Add the repository for Grafana stable releases:

echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list



Step 4: Update and Install Grafana:

Update the package list and install Grafana:

sudo apt-get update

sudo apt-get -y install grafana



Step 5: Enable and Start Grafana Service:

To automatically start Grafana after a reboot, enable the service:

sudo systemctl enable grafana-server



Start Grafana:

sudo systemctl start grafana-server



Step 6: Check Grafana Status:

Verify the status of the Grafana service to ensure it's running correctly:

sudo systemctl status grafana-server



You should see "Active (running)" in green colour

Press control+c to come out



Step 7: Access Grafana Web Interface:

The default port for Grafana is 3000

http://<monitoring-server-ip>:3000



Default id and password is "admin"

You can Set new password or you can click on "skip now".

Click on "skip now" (If you want you can create the password)



You will see the Grafana dashboard



10.2. Adding Data Source in Grafana

The first thing that we have to do in Grafana is to add the data source

Lets add the data source;



You can either click on "connections" in the left pane or click on "data sources" in the window to add the data source

Click on "Data sources" ----> Select "Prometheus" ----> Enable "default" toogle bar ----> Connection: Paste the Prometheus url ----> Remove / at the end of url ----> Scroll down and click on "Save and test" ----> If everything is fine, you will see "green" colour tick mark.



10.3. Adding Dashboards in Grafana

Click on "Dashboards" (left pane) ----> Here we have to add the Grafana dashboard. but as we dont know, we have to get the template of Grafana dashboard. To get the template ----> Goto browser and search for "Grafana node exporter dashboard" (URL: https://grafana.com/grafana/dashboards/1860-node-exporter-full/) ----> In the left pane, click on "Copy to clipboard" ----> Goto grafana ----> In the top right side, click on + ----> Import dashboard ----> Paste the id copied ----> Click on "Load" ----> Scroll down to see "Prometheus"  ----> Click on the dropdown ----> Select "Prometheus" ----> Clikc on "Import" ----> You can now see the dashboard ----> Click on "Save" icon in the top bar right side ----> Click on Save



Lets add another dashboard for Jenkins;

Goto browser and search for "Grafana jenkins dashboard" (URL: https://grafana.com/grafana/dashboards/9964-jenkins-performance-and-health-overview/) ----> In the left pane, click on "Copy to clipboard" ----> Goto grafana ----> In the top right side, click on + ----> Import dashboard ----> Paste the id copied ----> Click on "Load" ----> Scroll down to see "Prometheus"  ----> Click on the dropdown ----> Select "Prometheus" ----> Click on "Import" ----> You can now see the Jenkins dashboard ----> Click on "Save" icon in the top bar right side ----> Click on Save



Click on Dashboards in the left pane, you can see both the dashboards you have just added.



---

Lets setup ArgoCD using HELM;

---

Install HELM

curl -fsSL -o get\_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get\_helm.sh

./get\_helm.sh

helm version



Install ARGOCD using HELM

helm repo add argo https://argoproj.github.io/argo-helm

helm repo update



In production, it is always suggested to create a custom namespace ----> kubectl create namespace argocd ----> Lets install argocd in the namespace 'argocd' ----> helm install argocd argo/argo-cd --namespace argocd ----> kubectl get all -n argocd ----> You will see multiple things which are running ----> Under 'services' you can see 'argo-cd server' and the type as ClusterIP. But to access outside of the cluster, we need Load Balancer. So lets edit this ClusterIP to LoadBalancer ----> For this i will use patch ---->



EXPOSE ARGOCD SERVER:

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}' ----> kubectl get all -n argocd ----> Now you can see the service called 'argo-cd server' changed to Load Balancer instead of ClusterIP ----> Copy the load balancer url ----> This is one way of getting the loadbalancer url. Another way is to install "jq" (J Query) ---->



yum install jq -y



kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress\[0].hostname'

The above command will provide load balancer URL to access ARGO CD



Access the argocd using teh above load balancer url ----> Username: admin,



TO GET ARGO CD PASSWORD:

---

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

----> Copy the password and provide in argo cd console ----> You will see the argo cd console ----> Here is where we will do the k8s deployments ----> Click on 'New App' ----> App Name: ingress-app, Project name: default, Sync policy: Automatic, Source - Repo URL: <GitHub repo URL> ----> Revision: HEAD, Path: ./ ----> Cluster URL: Select the one from dropdown, Namespace: default (If you want to deploy the pods in custom namespace, create that namespace and provide here) ----> Create ----> Click on the 'ingress-app' ----> You can see the details of pods ----> kubectl get svc ----> Copy the load balancer url and access the app in new tab of browser ----> Lets say you want to change the image name in repo, you can change. Automatically you will see the modified app ----> Goto repo, change the image name, replicas count ----> Based on this everything will get updated in argocd ----> Goto Argocd ----> Click on 'sync' and 'snchronize' ----> Automatically the new app will get deployed. Here we dint used Jenkins but still we were able to automate the deployment process.



📄 Application Manifest for Argo CD

Save the below as argocd-app.yaml (outside the k8s/ directory, e.g., root of your repo or a separate argocd/ folder):



apiVersion: argoproj.io/v1alpha1

kind: Application

metadata:

  name: techsolutions-app

  namespace: argocd

spec:

  project: default

  source:

    repoURL: https://github.com/KastroVKiran/microservices-ingress.git

    targetRevision: HEAD

    path: k8s

  destination:

    server: https://kubernetes.default.svc

    namespace: default

  syncPolicy:

    automated:

      prune: true

      selfHeal: true

    syncOptions:

    - CreateNamespace=true

