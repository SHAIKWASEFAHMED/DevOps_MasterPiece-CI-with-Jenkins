
# DevOps-MasterPiece

In this project, I created an end-to-end CI-CD pipeline  while keeping in mind Securities Best Practices, DevSecOps principles and used all these tools *Git, GitHub , Jenkins,Maven, Junit, SonarQube, Jfrog Artifactory, Docker, Trivy, AWS S3, Docker Hub, Github CLI, EKS, ArgoCD, Prometheus, Grafana, Slack and Hashicorp Vault,*  to achive the goal.

#### I used Jenkins for Continuous Integration and ArgoCD for Continuous Deployment.

## Project Architecture
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/architecture.png)

## Pipeline flow:
1. When an event (commit) will occur in application code github repo ,  github webhook will push the code to Jenkins and Jenkins will start the build.

2. Maven will build the code, if the build fails, the whole pipeline will become a failure and Jenkins will notify the user using slack, If build success then

3.  Junit will do unit testing, if the application passes test cases then will go to the next step otherwise the whole pipeline will become a failure Jenkins will notify the user that your build fails.

4.  SonarQube scanner will scan the code and will send the report to the SonarQube server, where the report will go through the quality gate and gives the output to the web Dashboard.

 5. In the quality gate, we define conditions or rules like how many bugs or vulnerabilities, or code smells should be present in the code. 
                    Also, we have to create a webhook to send the status of quality gate status to Jenkins.
 If the quality gate status becomes a failure, the whole pipeline will become a failure then Jenkins will notify the user that your build fails.

5. After the quality gate passes, Artifacts will be send to Jfrog Artifactory. If artifacts send to artifactory successfully then will go to next stgae otherwise the whole pipeline will become a failure Jenkins will notify the user that your build fails.

6. After successful artifacts pushes to artifactory , Docker will build the docker image.
if the docker build fails when the whole pipeline will become a failure and Jenkins will notify the user that your build fails.

6. Trivy will scan the docker image, if it finds any Vulnerability then the whole pipeline will become a failure, and the generated report will be sent to s3 for future review and Jenkins will notify the user that your build fails.

7. After trivy scan docker images will be pushed to the docker hub, if the docker fails to push docker images to the docker hub then the pipeline will become a failure and Jenkins will notify the user that your build fails.

8. After the docker push, Jenkins will clone the kubernetes manifest repo from feature branch, if the repo is already present then it will only pull the changes. If jenkis unable to clone the repo then the whole pipeline will become a failure and Jenkins will notify the user that your build fails.

9. After Cloning the repo , Jenkins will update the imgae tag in deployment manifest. If jenkis unable to update the image tag  then the whole pipeline will become a failure and Jenkins will notify the user that your build fails.

10. After update image tag , jenkins will commit the change and push the code to feature branch. If jenkis unable to push the changes to feature branch  then the whole pipeline will become a failure and Jenkins will notify the user that your build fails.

11. After push changes to feature branch, jenkins will create a pull request against main branch.If jenkis unable to create pull request  then the whole pipeline will become a failure and Jenkins will notify the user that your build fails.

12. After pull request creation , a senior person from team will review and merge the pull request .

13. After merging feature branch into main branch , ArgoCD will pull the changes and deploy the application into kubernetes.

### PreRequisites
1. JDK 
1. Git 
1. Github
1. GitHub CLI
1. Jenkins
1. Sonarqube
1. Jfrog Artifactory
1. Docker
1. Trivy
1. AWS account
1. AWS CLI
1. Docker Hub account
1. Terraform
1. EKS Cluster
1. ArgoCD
1. Helm
1. Prometheus & Grafana
1. Hashicorp Vault
1. Slack

## Server Configuration ( I used )
1. 2 t2.medium ( ubuntu) EC2 Instances –  1. one for sonarqube and vault server
                                                     2. another for Jfrog Artifactory
1.  1 t2.large (ubuntu ) EC2 Instance -  For Jenkins, Docker, Trivy, AWS CLI, Github CLI, Terraform

1. EKS Cluster with t3.medium nodes


 # Want to create this Project by your own  then *Follow these  project steps*
## Step: 1 Installation Part 

### Stage-01 : Install JDK and Create a Java Springboot application
Push all the web application page code file into github

![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/code.png) 

### Stage-02 : Install Jenkins , Docker, Trivy, AWS CLI, Github CLI, Terrafrom ( t2.large )
#### Jenkins Installation Prerequisites  https://www.jenkins.io/doc/book/installing/linux/
1. Installation guide is available here  https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Jenkins_installation.md
1. After installation, install suggested plugins
1. Open Jenkins Dashboard and install required plugins – SonarQube Scanner, Quality gates Artifactory, Hashicorp Vault, Slack, Open Blue Ocean
1. go to manage jenkins > manage pulgins > search for plugins > Download now and install after restart
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/jenkins.png) 

#### Docker Installtion
1. Install docker using this command 
```sh
	sudo apt install docker.io
```
1.  add current user and jenkins user into docker group
```sh
 	sudo usermod -aG docker $USER
             sudo usermod -aG docker jenkins
```
#### Trivy Installation
1. Insatll trivy using this commands
```sh 
	sudo apt-get install wget apt-transport-https gnupg lsb-release
	wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
	echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a 		/etc/apt/sources.list.d/trivy.list
	sudo apt-get update
	sudo apt-get install trivy
```
#### AWS CLI installation
1. install aws cli  using this commands
```sh 
	sudo apt install unzip
            curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	unzip awscliv2.zip
	sudo ./aws/install
```
#### Install GitHub CLI
1. install github cli using this commands
```sh 
type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```
#### Install Terraform 
1. install teraaform using this commands
```sh 
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
gpg --no-default-keyring \
	--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
	--fingerprint
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt-get install terraform
```

### Stage-03 : Install SonarQube and Hashicorp Vault ( t2.medium )
#### SonarQube installation
1. install docker on sonarqube server
1. create a docker container to install SonarQube
```sh
    docker run -d -p 9000:9000 --name sonarqube sonarqube  
```
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/sonarqube.jpeg) 

   ``` 
####  Install Hashicorp Vault server 
HashiCorp Vault is a secret-management tool specifically designed to control access to sensitive credentials in a low-trust environment.
1. Installation vault using this commands
```sh
sudo curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64]  	https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
sudo apt install vault -y

```

 ### Stage-04: Install Jfrog Artifactory ( t2.medium )
1. install docker 
```sh
	sudo apt update
	sudo apt install docker.io
```
1. install Jfrog Artifactory
```sh 
	sudo docker pull docker.bintray.io/jfrog/artifactory-oss:latest
	sudo mkdir -p /jfrog/artifactory
	sudo chown -R 1030 /jfrog/
	sudo docker run --name artifactory -d -p 8081:8081 -p 8082:8082 \
	-v /jfrog/artifactory:/var/opt/jfrog/artifactory \
	docker.bintray.io/jfrog/artifactory-oss:latest

### Stage-05 : Install Slack
Slack is a workplace communication tool, “a single place for messaging, tools and files.” .

Install Slack from official website of Slack https://slack.com/intl/en-in/downloads/linux


### Stage-08: EKS Cluster Creation using Terraform
To create EKS Cluster using Terraform, I have put the terraform code here - https://github.com/praveensirvi1212/medicure-project/tree/master/eks_module

Suggestion – create eks cluster after successful configuration of jenkins server. When jenkins is able to creat pull request in manifest repo. 

# Done with Installation , Now will we integrate all the tools with Jenkins

### Stage-09: Install ArgoCD in EKS 
I am assuming that you have already kubernetes cluster running
1. use this commands to install argocd
```sh
kubectl create namespace argocd kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
1. edit the argocd-server service to NodePort to access argocd ui

### Stage-10: Install helm 
1. use this commands to install helm
```sh
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
### Stage-11: Install Prometheus and Grafana
1. use helm to install promethus and grafana
```sh
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm search repo prometheus-community
kubectl create namespace prometheus
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
kubectl get pods -n prometheus
kubectl get svc -n prometheus
#in order to make prometheus and grafana available outside the cluster, use LoadBalancer or NodePort instead of ClusterIP.
#Edit Prometheus Service
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
#Edit Grafana Service
kubectl edit svc stable-grafana -n prometheus
kubectl get svc -n prometheus

#Access Grafana UI in the browser using load balancero or nodeport

UserName: admin 
Password: prom-operator

refer this link for more info
https://www.coachdevops.com/2022/05/how-to-setup-monitoring-on-kubernetes.html

```


## Step: 2 Configure Individual tool

### Stage-01 : Jenkins Configuration
1. go to Manage Jenkins > configure tools > go to maven> give some name and click on install automatically
1. go to Manage Jenkins > configure tools > go to sonarqube scanner > give some name and click on install automatically 

### Stage-02 : Hashicorp Vault Configuration

I am assuming that your Vault server is installed and  running 
* open the ` /etc/vault.d/vault.hcl` file with vi or nano
* replace all the content of this file with with
```
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "raft_node_1"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
ui = true
```

* `sudo systemctl stop vault`
* `sudo systemctl start vault`

#### Commands to run to configure Vault and create AppRole

* `export VAULT_ADDR='http://127.0.0.1:8200'`
* `vault operator init`
	* ` copt the unseal tokens and initial root token , save it somewhere for later use `
* `vault operator unseal`
	* ` paste the first unseal token here`
* `vault operator unseal`
	* ` paste the second unseal token here`
* `vault operator unseal`
	* ` paste the third unseal token here`
* `vault login <Initial_Root_Token>`
   * `<Initial_Root_Token>` is found in the output of `vault operator init`

* `vault auth enable approle`
  	* https://www.vaultproject.io/docs/auth/approle
* `vault write auth/approle/role/jenkins-role token_num_uses=0 secret_id_num_uses=0 policies="jenkins"`
	* `this app role will use for jenkins integration` 
* `vault read auth/approle/role/jenkins-role/role-id`
	* copy the role_id and token, store somewhere
* `vault write -f auth/approle/role/jenkins-role/secret-id`
	* copy the secret-id and token , store somewhere



### Stage-03: SonarQube Server Configuration
1. Access the sonarqube ui using public ip of server and port 9000
1. default username-password `admin`  and ` admin`
##### Project setup
1.  create a project manually, give some name to project , project key
1. click on setup
1. click on other ci 
1. give some name to token and click on generate , save it for later username
1. click on global
1. select project type , in this case I used maven
1. copy the whole command and save it somewhere

##### Create Quality Gate and webhook
1.  Click on Quality Gates
1.  create a new Quality Gate accourding to your conditions
1.  click on Projects > click on all > select your project
1. set as default

##### Create webhook
1. click on Administration
1. click on Configuration > click on webhooks
1.  create a webhook > give some name
1. for url use `http://jenkins-server-url-with-port/sonarqube-webhook/`
1.  in secret leave the blank
1. click on create

##### Note: if this webhook not works fine , then recreate webhook after integrating the soanrqube with jenkins




### Stage-04: Artifactory Configuration
1. access the ui with public ip and port 8081:8081
1. use username-password as `admin` and `password`
1. update the password
1. create a maven repo
1. now go to Administration > user management > create a new user
1. now go to Repositories > create > locally > give some name like `my-local-repo`
 
### Stage-05: S3 Bucket creation to store Trivy report
1. login into aws account
1. search for S3
1. create a bucket with unique name

### Stage-06: DockerHub account creation
If you already have dockerhub account then no need to create another
1. go to dockerhub official website
1. click on sign up
1. fill the details and sing up
1. login into dockerhub

Note: we can create token from dockerhub to integrate jenkins but in this case I am using docker username and password.

### Stage-07: Slack Configuration
1.  go to this site https://slack.com/intl/en-in
1.  sign up with google
1.  create a workspace
1.  just give a name
1.  you can skip add team member
1.  give some name to team working ( channel name like - cicd-pipeline)
1. now go to this site https://praveensirvi.slack.com/apps
1. login into your workspace (if need)
1. now click on Get Essential Apps
1. search Jenkins in search bar > click on it 
1.  click on Add to Slack
1. select channel name
1. click on Add Jenkins CI Integration
1.  go down  to step 3 and copy Intergration Token Credential ID and save it somewhere
1. go down to Intergration Settings and copy the Token and save it somewhere
1. click on save settings



## Step: 3 Store the Credentials into Vault server
run all these commands into vault server
1. enable secrets path 
	 * `vault secrets enable -path=secrets kv`
1. write secret into secret path
     * `vault write secrets/creds/docker username=abcd password=xyz`

like wise we can store all the credentials into vault server . I have stored only docker credential
but you can store all your credential like this.

1. create a jenkins policy file with vi or nano ` jenkins-policy.hcl`
```sh
path "secrets/creds/*" {
 capabilities = ["read"]
}
```
policy is created with * means vault server can read credential from every path. No need to create policies for each path like `secrets/creds/docker` , `secrets/creds/slack` etc…

1. run this commands to create policy
* `vault policy write jenkins jenkins-policy.hcl`
 
 

## Step: 4 Integrate all the tools into jenkins 
### Stage-01: Hashicorp Vault server integration with Jenkins
1. go to jenkins >  Manage  Jenkins >Manage Credentials > system > Add credentials > Vault App Role Credentials > paste role-id and secret-id token (we create in Vault - approle)  and save and apply.
1. now to  Manage  Jenkins  >  configure system / system >  search for vault plugin
1. give the url of vault server
1. attach the credentials we created
1. click on Advanced
1. select k/v engine as 1
1. click on skip ssl verification
1. Apply and Save

### Stage-02: SonarQube server integration with Jenkins
1.  go to Manage jenkins >Manage Credentials > system > Add credentials > secret text file > paste token we create in sonarqube and save and apply.
1. now to  Manage  Jenkins  >  configure system / system >  search for SonarQube Server
1. enable the environment  variables
1. write the name for sonaqube server
1. paste the url of SonarQube server
1. select the credential
1. Apply and Save

### Stage-03: Jforg Artifactory integration with Jenkins
1. go to Manage jenkins >Manage Credentials > system > Add credentials > username and password > write the username and password we created in jforg artifacotry and save and apply.
1. now to  Manage  Jenkins  >  configure system / system >  search for Jfrog
1. give instance id as artifactory name
1. Jfrog Platform url - artifactory url like - http://localhost:8082
1. click on Advanced
1. JFrog Artifactory URL - http://localhost:8082/artifactory
1. JFrog Distribution URL - http://localhost:8082/distribution
1. Default Deployer Credentials – give username and password of artifactory (not admin user)
1. Apply and save
### Stage-04:  AWS S3 integration with Jenkins
1. for S3 integration we will configure aws cli in pipeline itself
1. create credentials for aws cli , use both as secret  text

### Stage-05:  DockerHub integration with Jenkins
1. go to Manage jenkins >Manage Credentials > system > Add credentials > vault Username-Password Credential
1. namespace – leave blank
1. Prefix Path – leave blank
1. Path – secrets/creds/docker
1. Username Key- username
1. Password key – password
1. k/v engine – 1
1. id – give some id like docker-cred
1. Description -  give some description
1. click on Test Vault Secrets retrieval  > should give output as secrets retrive successfully otherwise reconfigure vault server in jenkins
1. Apply and Save

### Stage-05:  Slack integration with Jenkins
1.  go to Manage jenkins >Manage Credentials > system > Add credentials > secret text file > give some name to credentials , paste token we create in slack app and save and apply.
1. now to  Manage  Jenkins  >  configure system / system >  search for Slack
1. workspace – your workspace name ( you create after the login into slack)
1. Credential – attach slack token
1. Default channel name – write channel name we created at the time of slack installation like - #cicd-pipeline
1. Apply and save

### Stage-06:  GitHub integration with Jenkins ( application code repo )
1. go to github  > go to application code repo > settings
1. go to webhook >  Add webhook
1. Payload URL -  http://jenkins-server-public-ip-with-port/github-webhook/
1.  click on Add webhook


### Stage-07:  ArgoCD integration with Github ( k8s manifest repo)
1. access the argocd UI – node public ip and node port
1. user username as `admin`
1. for password run this command
```sh
 kubectl -n argocd get secret argocd-initial-admin-secret -o yaml
```
1. copy the password and decode it using
```sh
ehco “copied-password” | base64 -d
```
1. copy the decoded password and login into argocd
1.  go to User Info – update password
1.  now go to Application
1. click on New Application
1. give app name
1. chose Project Name as default
1. SYNC Policy – Automactic
1. enable PRUNE RESOURCES and SELF HEAL
1. SOURCE-
1. Repository URL – give your repo url where you stored k8s manifest
1. Path – yamls
1. DESTINATION -
1. Cluster Url – chose default
1. namespace- default
1. click on create

### Stage-08:  Prometheus and Grafana Integration
use this docs to import Dashboard into grafana
https://www.coachdevops.com/2022/05/how-to-setup-monitoring-on-kubernetes.html









# We integrated all the tools with Jenkins, Now Create a declarative jenkins  pipeline for each stage.

## Step: 3 Pipeline creation

### General Jenkins  declarative Pipeline Syntax


```sh 
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1' 
    }
    stages {
        stage('Example') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

### Stage-01 : Git Checkout 
1. Define a stage as git checkout
1. go to this site https://opensource.triology.de/jenkins/pipeline-syntax/
1. search for checkout: check out version control
1. give your github url, branch and generate the pipeline synatx
1. paste it into stage steps git check

```sh 
stage('Checkout git') {
            steps {
              git branch: 'main', url:'https://github.com/praveensirvi1212/DevOps_MasterPiece-CI-with-Jenkins.git'
            }
        }
```
### Stage-02 : Build and Junit test
1. Define a stage as Build and Junit test 
1. go to this site https://opensource.triology.de/jenkins/pipeline-syntax/
1. search for sh:shell script 
1. give your shell comman and generate the pipeline synatx
1. paste it into stage >  steps > sh ‘ shell command’


```sh
stage ('Build & JUnit Test') {
	steps {
		sh 'mvn install' 
	}
	
}
```
### Stage-03 : SonarQube Analysis
In this stage i used withSonarQubeEnv to  Prepare SonarQube Scanner environment
and shell command sh
1. Define  a stage SonarQube Analysis
1. paste the command that we created at the time of sonarqube project creation
```sh
stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('SonarQube-server') {
                        sh '''mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=gitops-with-argocd \
                        -Dsonar.projectName='gitops-with-argocd' \
                        -Dsonar.host.url=$sonarurl \
                        -Dsonar.login=$sonarlogin'''
                }
            }
        }

```

### Stage-04 : Quality gate
This step pauses Pipeline execution and wait for previously submitted SonarQube analysis to be completed and returns quality gate status. Setting the parameter abortPipeline to true will abort the pipeline if quality gate status is not green.
1. Define a stage as Quality gate
1. go to this site https://opensource.triology.de/jenkins/pipeline-syntax/
1. search for  waitForQualityGate: Wait for SonarQube analysis to be completed and return quality gate status
1. generate pipeline syntax and paste it into steps
1. timeout is optional 
```sh
stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
```
### Stage-05 : Jfrog Artifactory
1. We wrap the steps in a script block to execute them as a Groovy script.
2. Inside the try block, we create a reference to the Artifactory server using the Artifactory.newServer method. You need to provide the URL of your Artifactory instance and the credentials ID configured in Jenkins.
3. We define an uploadSpec as a JSON string, specifying the pattern of files to upload and the target repository in Artifactory.
4. We call the server.upload(uploadSpec) method to upload the files to Artifactory based on the specified upload specification.
5. If any exception occurs during the upload process, the catch block will be executed, and an error message will be displayed.

```sh
steps {
                script {
                    try {
                        def server = Artifactory.newServer url: 'http://13.232.95.58:8082/artifactory', credentialsId: 'jfrog-cred'
                        def uploadSpec = """{
                            "files": [
                                {
                                    "pattern": "target/*.jar",
                                    "target": "${TARGET_REPO}/"
                                }
                            ]
                        }"""
                        
                        server.upload(uploadSpec)
                    } catch (Exception e) {
                        error("Failed to deploy artifacts to Artifactory: ${e.message}")
                    }
                }
            }
        }
```
### Stage-06 : Docker Build
First write your dockerfile to build docker images.I have posted my  application repo code
In this stage i  used shell command sh to build docker image
1. Define  a stage Docker Build
1. go to this site https://opensource.triology.de/jenkins/pipeline-syntax/
1. search for sh:shell script 
1. give your shell command to build image > generate pipeline syntax
1. I used build id of jenkins and git commit id for versions of docker images

```sh
sstage('Docker  Build') {
            steps {
               
      	         sh 'docker build -t ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT} .'
                
            }
        }
```
### Stage-06: Trivy Image scan
In this stage i  trivy shell command sh to scan docker image
1. Define  a stage Trivy Image scan
1. go to this site https://opensource.triology.de/jenkins/pipeline-syntax/
1. search for sh:shell script 
1.  give your Trivy shell command to scan build image
#### Note – There are 3 types of report output  format of trivy ( Table , JSON, Template). I used  html template for output report of trivy scan
```sh
stage('Image Scan') {
            steps {
      	        sh ' trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT} '
            }
        }
	}
}
```
### Stage-07: Upload report generated by trivy to AWS S3
In this stage i  shell command sh to scan docker image
1. Define  a stage Upload report to AWS S3
1. first create a AWS s3 bucket 
1. go to this site https://opensource.triology.de/jenkins/pipeline-syntax/
1. search for sh:shell script
1. give your shell command to upload object to aws s3

```sh
stage('Upload Scan report to AWS S3') {
              steps {
                  
                //  sh 'aws configure set aws_access_key_id "$AWS_ACCESS_KEY_ID"  && aws configure set aws_secret_access_key "$AWS_ACCESS_KEY_SECRET"  && aws configure set region ap-south-1  && aws configure set output "json"' 
                  sh 'aws s3 cp report.html s3://devops-mastepiece/'
              }
        }
```

### Stage-08: Push Docker images to DockerHub
In this stage i  shell command sh to push docker image to docker hub. I stored Credentials into Vault and access into jenkins using  vault key. You can store DockerHub credentials into jenkins and use as environment variables
1. Define  a stage Docker images push
1. go to this site https://opensource.triology.de/jenkins/pipeline-syntax/
1. search for sh:shell script
1. give your shell command to push docker images to docker hub

 ``` sh
stage ('Docker Build') {
            steps {
                withVault(configuration: [skipSslVerification: true, timeout: 60, vaultCredentialId: 'vault-token', vaultUrl: 'http://13.232.53.209:8200'], vaultSecrets: [[path: 'secrets/creds/docker', secretValues: [[vaultKey: 'username'], [vaultKey: 'password']]]]) {
                    
                    sh "docker login -u ${username} -p ${password} "
                    sh 'docker push ${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}'
                    
                    
                }
            }
        }
 ``` 
### Stage-08: Clone/Pull Repo (k8s manifest repo )
1. in this stage first we check whether repo is already exists or not
1. if exists then pull the changes
1. if not then will clone the repo

 ```sh
stage('Clone/Pull Repo') {
            steps {
                script {
                    if (fileExists('DevOps_MasterPiece-CD-with-argocd')) {

                        echo 'Cloned repo already exists - Pulling latest changes'

                        dir("DevOps_MasterPiece-CD-with-argocd") {
                          sh 'git pull'
                        }

                    } else {
                        echo 'Repo does not exists - Cloning the repo'
                        sh 'git clone -b feature https://github.com/praveensirvi1212/DevOps_MasterPiece-CD-with-argocd.git'
                    }
                }
            }
        }
```

# Stage: 10 Update Manifest
1. used sed command to replace images tag in deployment manifests
```sh
stage('Update Manifest') {
            steps {
                dir("DevOps_MasterPiece-CD-with-argocd/yamls") {
                    sh 'sed -i "s#praveensirvi.*#${IMAGE_REPO}/${NAME}:${VERSION}-${GIT_COMMIT}#g" deployment.yaml'
                    sh 'cat deployment.yaml'
                }
            }
        }
```
## Stage: 11 Commit and Push Changes to k8s manifest repo
1. set the global username
1. set the remote repo url
1. checkout the branch to feature
1. stage the chages
1. commit the changes
1. push the changes to feature branch
```sh
stage('Commit & Push') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    dir("DevOps_MasterPiece-CD-with-argocd/yamls") {
                        sh "git config --global user.email 'praveen@gmail.com'"
                        sh 'git remote set-url origin https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}'
                        sh 'git checkout feature'
                        sh 'git add deployment.yaml'
                        sh "git commit -am 'Updated image version for Build- ${VERSION}-${GIT_COMMIT}'"
                        sh 'git push origin feature'
                    }
                }    
            }
        }
```

## Stage: 12 Create Pull Request
Reason to create a pull request is that argocd is sync automatically with github . Github is only single source of truth for argocd. So if jenkins push the chaanges to main branch then argocd will deploy changes directly without review the changes. This should not happen in Production environment . That’s why we create pull request against main branch. So a senior person form team can review the changes and merge it to main branch. Then n then only changes will go to production environment. 
1. 
```sh
stage('Raise PR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    dir("DevOps_MasterPiece-CD-with-argocd/yamls") {
                        sh '''
                            set +u
                            unset GITHUB_TOKEN
                            gh auth login --with-token < token.txt
                            
                        '''
                        sh 'git branch'
                        sh 'git checkout feature'
                        sh "gh pr create -t 'image tag updated' -b 'check and merge it'"
                    }
                }    
            }
        } 
```



## Stage: 13 Post build action 
In post build action i used slack notification . After  build jenkins will send notification massage to slack whether your build success or failed.
1. go to jenkins > your project > pipeline syntax > search for slacksend: send slack message 
1. write your channel name and message > generate pipeline synatx .
#### Note – i used custom messages for my project . I Created a function for slack notification and called the function into post build .
 ```sh
post{
	always{
		sendSlackNotifcation()
	}
}
```
sendSlackNotification function 
 ```sh
def sendSlackNotifcation()
{
if ( currentBuild.currentResult == "SUCCESS" ) {
buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *SUCCESS*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL} \n"
slackSend( channel: "#devops", token: 'slack-token', color: 'good', message: "${buildSummary}")
}
else {
buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *FAILURE*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL}\n \n "
slackSend( channel: "#devops", token: 'slack-token', color : "danger", message: "${buildSummary}")
}
}
 ```
#### Find whole pipeline here
 https://github.com/praveensirvi1212/DevOps_MasterPiece-CI-with-Jenkins/blob/main/Jenkinsfile

## Step: 4 Projecct Output

# Final outputs of this Project
### Jenkins Output : 
After 86th  Build my  jenkins pipeline became successful. 
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/pipelineop.png) 

### Sonarqube Output: 
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/sonarqubeop.png) 

### Quality Gate Status in Jenkins
This Output is the build number 86th. SonarQube Quality gate status is green and passed .   
You applied your custom quality gate like : there should be zero ( bug, Vulnerability , code smell ) and your code have greater then 0 (bugs, vulnerability , code smells) . Then your quality gate status will become failure or red. If your quality gate status beome failure , stages after quality gate will be failure.
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/qualitygateop.png) 

### Trivy report in AWS S3 push by jenkins
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/trivy-report-s3.png) 


### Trivy report 
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/TrivyReprt.png) 


### Images in DockerHub pushed by jenkins 
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/dockerhubop.png) 

### kubernetes output ( deployment and service created by jenkins) 
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/kubernetesop.png) 

### Application output deployed in k8s 
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/spring-boot-app-op.png) 

### Slack output 
![](https://github.com/praveensirvi1212/DevSecOps-project/blob/main/Images/slackop.png) 

