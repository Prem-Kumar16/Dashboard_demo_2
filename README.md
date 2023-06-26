# Dashboard_demo_2

The repository contains the instructions and code that allows you to create a kubernetes cluster for dashboard demo using virtual can interface

## Overview

The below image is the detailed overview of the following demo

<img width="582" alt="Screenshot 2023-06-20 220215" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/d1b1d8db-8cf8-4bf1-89d3-ccf3adedd0c3">


## THIS DEMO WILL WORK BEST ON MUMBAI (ap-south-1) region

Deploy EC2 in Mumbai region

Please open the below link in new tab to ease the process

[![Launch](https://samdengler.github.io/cloudformation-launch-stack-button-svg/images/ap-south-1.svg)](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/quickcreate?templateURL=https%3A%2F%2Fs3.ap-south-1.amazonaws.com%2Fcf-templates-fui01m96flo3-ap-south-1%2F2023-06-16T080737.467Zxme-demo-2-dashboard-ec2-template.yml&stackName=dashboard-demo-2-SDV-stack)

Acknowledge the creation of the stack and press the button **Create stack** on the bottom right. 

The ```dashboard-demo-2-SDV-stack``` CloudFormation stack will take about **2 minutes** to be created.

### Locally downloading the Private key file

Follow the steps below to download the private .pem key file to SSH into the instance

Open cloudshell and run the following command

```sh 
aws ec2 describe-key-pairs --filters Name=key-name,Values=keypair-for-ewaol --query KeyPairs[*].KeyPairId --output text
```

The output will be the key ID. Note down it

Run the below command to save .pem file in the cloudshell directory

Change the keyid paramater to the output of previous command

```sh
aws ssm get-parameter --name /ec2/keypair/<keyid here> --with-decryption --query Parameter.Value --output text > keypair-for-ewaol.pem
```

<img width="911" alt="Screenshot 2023-06-16 085413" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/502fba9f-99d6-4a10-a806-df7239149409">

Go to actions -> download file and paste this path "/home/cloudshell-user/keypair-for-ewaol.pem" inside the path field to save .pem key file locally

<img width="516" alt="Screenshot 2023-06-16 090012" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/58580da4-1891-44c1-8f33-a968324f8a4e">

If you go to ec2 instances page, you will find two newly created instances named "Dashboard-demo-master" & "Dashboard-demo-worker". SSH into the instances using the key file  that you have previously downloaded

#### Important note, while SSH change the user name from root to ewaol i.e., instead of root@ip.eu-central-1.compute.amazonaws.com, you should use ewaol@ip.eu-central-1.compute.amazonaws.com

### Do the below steps in MASTER ec2 instance (Dashboard-demo-master)

After connected to the instance via SSH, run the below commands in master ec2 instance

Note : Give the access & secret access key to the instance via "aws configure" command & the default region name must be ap-south-1

```sh
aws configure
cd Dashboard_demo_2/datasimulator/
```

Now we need to change the spec.nodeName inside the deployaws.yaml file of both the datasimulator & dashboard folder. Follow the steps below to do that

Run the below command 

```sh
sudo kubectl get nodes
```

From the output of the above command, copy the node name 

Now run the below command 

```sh
sudo vi deployaws.yaml
```

Inside the vi editor, change the value of nodeName spec (reference image below) to the value that you have previously copied via get node command. (The value is nothing but your master ec2 instance's private ip address)

<img width="374" alt="Screenshot 2023-06-16 121549" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/f9ab1104-e339-44cb-9047-a2661084b886">

Run the below steps to log into ECR Repository for pulling images from it

```sh
AWS_DEFAULT_REGION=ap-south-1
PASS=$(aws ecr get-login-password --region $AWS_DEFAULT_REGION)
```

Create a kubectl secret 

```sh
sudo kubectl create secret docker-registry regcred \
   --docker-server=783584839454.dkr.ecr.ap-south-1.amazonaws.com \
   --docker-username=AWS \
   --docker-password=$PASS
```

The below command helps you to run the pod

```sh
sudo kubectl apply -f deployaws.yaml -f service.yaml
```

If you want to know the status of pods, the below commands will help you

```sh
sudo kubectl get pods
sudo kubectl describe pods
```

In your browser, open a new tab and open http://3.7.144.155:3000/

You should see a web page similar to the image below

<img width="960" alt="Screenshot 2023-06-16 122718" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/6bbc22e8-654a-476a-9e90-5248e3f7a25f">

Run the below command and copy the output which contains the k3s token to link master and worker node 

```sh
sudo cat /var/lib/rancher/k3s/server/node-token
```

<img width="778" alt="Screenshot 2023-06-16 124600" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/cf52a0fb-0c4b-4bc6-b50e-6dc21714f668">

### Do the below steps in WORKER ec2 instance (Dashboard-demo-worker)

Run the command below to link worker node with master node

```sh
sudo k3s-agent -t <your k3s token here> -s https://3.7.144.155:6443
```

<img width="938" alt="Screenshot 2023-06-16 125507" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/1b3d6b00-9763-4408-b7c0-05a5dd35d351">

### Now go back to MASTER ec2 instance (Dashboard-demo-master)

Run the below command to check whether the nodes are linked, if success copy the worker node name (whose role is <none>)

```sh
sudo kubectl get nodes
```

You should see an output similar to the below image

<img width="566" alt="Screenshot 2023-06-16 125812" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/1e7728bf-6b72-481a-965a-f7939864dd2b">

Now run the below command to open the vi editor of deployaws.yaml file inside the dashboard folder to change the value of nodeName spec (reference image below) to the value that you have previously copied via get node command. (The value is nothing but your worker ec2 instance's private ip address)

```sh
cd ..
cd dashboard/
sudo vi deployaws.yaml
```

<img width="419" alt="Screenshot 2023-06-16 130742" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/1589551a-0684-49b3-9802-8889a98648f6">

Now run the command to start the service and pod

```sh
sudo kubectl apply -f deployaws.yaml -f service.yaml
```

Run the below command to check whether the pods is running in correct node

```sh
sudo kubectl get pods -o wide
```

The output should be similar to the below image

<img width="879" alt="Screenshot 2023-06-16 131736" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/1cd87f4a-5b0b-4005-a36f-cf93654f3bae">

Open a new tab in your browser and open http://13.127.34.56:4000/. You should see a webpage similar to the below image

<img width="389" alt="Screenshot 2023-06-16 132153" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/90cf2e3d-5282-4349-8064-f7168cc32c8a">

Now you can change any values in can bus input simulator and see the changes in the digital cockpit

<img width="960" alt="Screenshot 2023-06-16 132441" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/f4cd5941-4d53-433b-8a30-2274bc7e5d0b">

### Cleanup

From [CloudFormation](https://console.aws.amazon.com/cloudformation/home) just delete `dashboard-demo-2-SDV-stack` stack.

Do not forget to delete the downloaded keypair in cloudshell by running the below command

```sh
rm keypair-for-ewaol.pem
```
