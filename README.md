# Dashboard_demo_2

The repository contains the instructions and code that allows you to create a kubernetes cluster for dashboard demo using virtual can interface

## Overview

The below image is the detailed overview of the following demo

<img width="582" alt="Screenshot 2023-06-20 220215" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/d1b1d8db-8cf8-4bf1-89d3-ccf3adedd0c3">


## THIS DEMO WILL WORK BEST ON MUMBAI (ap-south-1) region

Deploy EC2 in Mumbai region

Please open the below link in new tab to ease the process

[![Launch](https://samdengler.github.io/cloudformation-launch-stack-button-svg/images/ap-south-1.svg)](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/quickcreate?templateURL=https%3A%2F%2Fs3.ap-south-1.amazonaws.com%2Fcf-templates-fui01m96flo3-ap-south-1%2F2023-06-29T125651.979Z84n-demo-2-dashboard-ec2-template.yml&stackName=dashboard-demo-2-SDV-stack)

Acknowledge the creation of the stack and press the button **Create stack** on the bottom right. 

The ```dashboard-demo-2-SDV-stack``` CloudFormation stack will take about **2 minutes** to be created. This cloudformation stack creates two ec2 instances named "Dashboard-demo-master" & "Dashboard-demo-worker" to deploy the demo, a security group, a key pair and it also associates the ec2 instances with the already created elastic IPs.

### Locally downloading the Private key file

Follow the steps below to download the private .pem key file to SSH into the instance

Open cloudshell and run the following command

```sh 
aws ec2 describe-key-pairs --filters Name=key-name,Values=keypair-for-ewaol-demo2 --query KeyPairs[*].KeyPairId --output text
```

The output will be the key ID. Note down it

Run the below command to save .pem file in the cloudshell directory

Change the keyid paramater to the output of previous command

```sh
aws ssm get-parameter --name /ec2/keypair/<keyid here> --with-decryption --query Parameter.Value --output text > keypair-for-ewaol-demo2.pem
```

<img width="960" alt="Screenshot 2023-06-29 195054" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/2e11c1ce-b36f-4268-a608-1b731b3324e5">


Go to actions -> download file and paste this path "/home/cloudshell-user/keypair-for-ewaol-demo2.pem" inside the path field to save .pem key file locally

<img width="456" alt="Screenshot 2023-06-29 195242" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/a20936e2-5085-4f3b-a7ff-036fc899876f">

If you go to ec2 instances page, you will find two newly created instances named "Dashboard-demo-master" & "Dashboard-demo-worker". SSH into the instances using the key file  that you have previously downloaded

#### Important note, while SSH change the user name from root to ewaol i.e., instead of root@ip.eu-central-1.compute.amazonaws.com, you should use ewaol@ip.eu-central-1.compute.amazonaws.com

### Do the below steps in MASTER ec2 instance (Dashboard-demo-master)

After connected to the instance via SSH, run the below commands in master ec2 instance

Note : Give the access key ID & secret access key to the instance via "aws configure" command & the default region name must be ap-south-1

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

Paste the node name you copied from the last command to the below command and run it (The value is nothing but your master ec2 instance's private ip address)

```sh
sudo sed -i 's/ip-172-31-11-89/<nodeName here>/g' deployaws.yaml
```

After the aws configuration step, launch the below cloudformation stack which will create all the necessary developer tools and creates a whole codepipeline for data simulator webpage.

Please open the below link in new tab to ease the process

[![Launch](https://samdengler.github.io/cloudformation-launch-stack-button-svg/images/ap-south-1.svg)](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/quickcreate?templateURL=https%3A%2F%2Fs3.ap-south-1.amazonaws.com%2Fcf-templates-fui01m96flo3-ap-south-1%2F2023-07-28T030913.668Zm64-codepipeline_cfn_template_demo2_datasim.yaml&stackName=codepipeline-datasim-demo2)

Acknowledge the creation of the stack and press the button Create stack on the bottom right.

The ```codepipeline-datasim-demo2``` CloudFormation stack will take about 2 minutes to be created.

After the creation of the stack, go to the codePipeline management console, You will see a pipeline named **autodatasimulatordemocodepipeline** created. Wait for it to be completed.

If you want to know the status of pods, the below commands will help you

```sh
sudo kubectl get pods
sudo kubectl describe pods
```

In your browser, open a new tab and open http://3.7.144.155:3000/ (This is the Elastic IP where your ec2 instance consisting the demo will be running)

You should see a web page similar to the image below

<img width="960" alt="Screenshot 2023-06-16 122718" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/d756a227-d958-485a-840f-361aa2d22323">

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

Run the below command to check whether the nodes are linked, if success copy the worker node name (whose role is "<none>")

```sh
sudo kubectl get nodes
```

You should see an output similar to the below image

<img width="566" alt="Screenshot 2023-06-16 125812" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/1e7728bf-6b72-481a-965a-f7939864dd2b">

Now run the below command inside the dashboard folder to change the value of nodeName spec (reference image below) to the value that you have previously copied via get node command. (The value is nothing but your worker ec2 instance's private ip address)

```sh
cd ..
cd dashboard/
sudo sed -i 's/ip-172-31-1-113/<nodeName here>/g' deployaws.yaml
```

<img width="419" alt="Screenshot 2023-06-16 130742" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/1589551a-0684-49b3-9802-8889a98648f6">


After this step, launch the below cloudformation stack which will create all the necessary developer tools and creates a whole codepipeline for dashboard webpage.

Please open the below link in new tab to ease the process

[![Launch](https://samdengler.github.io/cloudformation-launch-stack-button-svg/images/ap-south-1.svg)](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/quickcreate?templateURL=https%3A%2F%2Fs3.ap-south-1.amazonaws.com%2Fcf-templates-fui01m96flo3-ap-south-1%2F2023-07-28T034339.512Z2g5-codepipeline_cfn_template_demo2_dash.yaml&stackName=codepipeline-dash-demo2)

Acknowledge the creation of the stack and press the button Create stack on the bottom right.

The ```codepipeline-dash-demo2``` CloudFormation stack will take about 2 minutes to be created.

After the creation of the stack, go to the codePipeline management console, You will see a pipeline named **autodashboarddemocodepipeline** created. Wait for it to be completed.

<img width="662" alt="Screenshot 2023-07-28 091559" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/4cea44e2-54b7-4876-81f1-f9ec2f2e0bb3">


Run the below command to check whether the pods is running in correct node

```sh
sudo kubectl get pods -o wide
```

The output should be similar to the below image

<img width="879" alt="Screenshot 2023-06-16 131736" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/1cd87f4a-5b0b-4005-a36f-cf93654f3bae">

Open a new tab in your browser and open http://13.127.34.56:4000/. You should see a webpage similar to the below image

<img width="389" alt="Screenshot 2023-06-16 132153" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/90cf2e3d-5282-4349-8064-f7168cc32c8a">

Now you can change any values in can bus input simulator and see the changes in the digital cockpit

#### Note : The values below right, left and main circle accepts only 0,1 & 2 values. If you try to give value other than this, it will throw error message (refer the image below)

![Screenshot (139)](https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/2e20fb20-e6f2-4137-ac1e-5c73a3d362ab)

Final output image : 

<img width="960" alt="Screenshot 2023-06-16 132441" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/5a047d30-cb1a-45ec-a890-933bac58fc69">

### Cleanup

From [CloudFormation](https://console.aws.amazon.com/cloudformation/home) delete `codepipeline-datasim-demo2` stack.

If the delete is failed, go to the resources tab under `codepipeline-datasim-demo2` stack and open the ECR Repository in a new tab.

<img width="367" alt="Screenshot 2023-07-27 070006" src="https://github.com/Prem-Kumar16/Dashboard_demo_1/assets/75419846/4f20c65e-81b1-47e3-9d02-40e1d1919af3">


Choose delete stack again. This time a message appears to retain the resources. Choose to retain and click delete.

<img width="473" alt="Screenshot 2023-07-28 080817" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/5f600728-927c-4055-8b97-3aedabaa330e">


<img width="451" alt="Screenshot 2023-07-28 080937" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/471f1570-e5c6-4ff9-9ca9-120f38b02577">


Now the `codepipeline-datasim-demo2` stack will be deleted.

Go to the ECR Repo page that you have already opened in new tab and delete the **autodatasimulatordemo** repo.

Do the same for deleting `codepipeline-dash-demo2` stack.

Finally from [CloudFormation](https://console.aws.amazon.com/cloudformation/home) just delete `dashboard-demo-2-SDV-stack` stack.

Do not forget to delete the downloaded keypair in cloudshell by running the below command

```sh
rm keypair-for-ewaol-demo2.pem
```
