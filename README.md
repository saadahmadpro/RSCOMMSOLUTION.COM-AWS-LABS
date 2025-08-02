# RSCOMMSOLUTION.COM-AWS-LABS

NEW LAB 2

Create 2 Public EC2 (ubuntu 22.04) with MyIP Access Security Group and pass docker installation script in userdata.


Create an ECR Registry


Attach ECR Full access IAM Role to both EC2s


Connect to 1st Machine and copy website file (S) and create a docker file 


Build docker image and push it to ECR


Connect to second machine , pull the image from ECR and run a container with that image




Got it! Here's a single bash script you can run on your Ubuntu 22.04 EC2 instance to:
Install AWS CLI v2


Configure AWS CLI with your credentials (you'll need to provide them interactively or set them as environment variables)


Create an ECR repository


Set up Docker


Log in to ECR


Pull and run a Docker image (assuming you have one)


This script covers the basic setup for your lab tasks:
#!/bin/bash

# Update packages and install dependencies
sudo apt update -y
sudo apt install -y unzip curl docker.io

# Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws

# Verify AWS CLI installation
echo "AWS CLI version:"
aws --version

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add ubuntu user to docker group (optional, for running docker without sudo)
sudo usermod -aG docker $USER

echo "Docker version:"
docker --version

# Configure AWS CLI (you can automate this by setting env vars, but here is manual prompt)
echo "Configure AWS CLI now:"
aws configure

# Variables
ECR_REPO_NAME="my-ecr-website"
AWS_REGION=$(aws configure get region)

if [ -z "$AWS_REGION" ]; then
  echo "AWS region not set. Please set the default region with 'aws configure'. Exiting."
  exit 1
fi

# Create ECR repository
echo "Creating ECR repository: $ECR_REPO_NAME"
aws ecr create-repository --repository-name "$ECR_REPO_NAME" --region "$AWS_REGION"

# Get AWS account ID for ECR login
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Log in to ECR
echo "Logging into ECR"
aws ecr get-login-password --region "$AWS_REGION" | docker login --username AWS --password-stdin "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"

echo "Setup complete. You can now build, tag, and push your Docker image."

# Example: Build Docker image (optional)
# docker build -t my-app .

# Tag Docker image for ECR (replace with your image name)
# docker tag my-app:latest "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:latest"

# Push Docker image to ECR (optional)
# docker push "$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO_NAME:latest"

echo "Done. Remember to reconnect to your instance or logout/login to apply Docker group changes."


Notes:
You will be prompted to enter your AWS Access Key, Secret Key, and region during the aws configure step.


This script installs the latest AWS CLI v2, not the old v1 from apt repos.


Docker group add requires logout/login to take effect.


Modify the script to add your Docker build and push commands as needed.

Great! Now that your environment is set up, here’s the next sequence of steps to complete your lab tasks:

1. Build and Push Docker Image to ECR
To complete the Docker setup, you’ll need to:
Create a Dockerfile for your web application (or any application you want to containerize).


Build the Docker image.


Tag the image for ECR.


Push the image to your ECR repository.


Here's a general outline:
Create Dockerfile (for example, for a simple Node.js app):

 Create a Dockerfile for your application (you can use any app; here's a simple one for a static web server):
 Use official Nginx image from Docker Hub
FROM nginx:latest

# Copy your static website files to Nginx
COPY ./website /usr/share/nginx/html

# Expose port 80
EXPOSE 80




How to use:
Copy the above code to a file on your EC2 instance, e.g., setup_lab.sh.


Run chmod +x setup_lab.sh to make it executable.


Run ./setup_lab.sh.


Follow prompts for AWS CLI configuration.


After script finishes, you have AWS CLI and Docker set up, and an ECR repo created.


You can now build, tag, and push Docker images to ECR.
























Great! Now that your environment is set up, here’s the next sequence of steps to complete your lab tasks:
1. Build and Push Docker Image to ECR
To complete the Docker setup, you’ll need to:
Create a Dockerfile for your web application (or any application you want to containerize).


Build the Docker image.


Tag the image for ECR.


Push the image to your ECR repository.


Here's a general outline:
Create Dockerfile (for example, for a simple Node.js app):

 Create a Dockerfile for your application (you can use any app; here's a simple one for a static web server):

 # Use official Nginx image from Docker Hub
FROM nginx:latest

# Copy your static website files to Nginx
COPY ./website /usr/share/nginx/html

# Expose port 80
EXPOSE 80


Build the Docker Image:

 If your Dockerfile is in the current directory, build the image:

 docker build -t my-ecr-website .


Tag the Docker Image for ECR:

 After the image is built, tag it with the ECR repository URL (replace with your region and account):

 docker tag my-ecr-website:latest $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest


Push the Image to ECR:

 Push the tagged image to your ECR repository:

 docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest



2. Pull the Docker Image from ECR on Another EC2 (Second Machine)
Login to ECR (on the second EC2 instance):

 Use the AWS CLI to authenticate Docker to your ECR:

 aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com


Pull the Docker Image:

 Once you’re logged in, pull the image:

 docker pull $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest


Run the Docker Container:

 After pulling the image, run it on your second EC2 instance:

 docker run -d -p 80:80 $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest


This will run the container in detached mode (-d), binding port 80 of the EC2 instance to port 80 inside the container.

3. Verify Your Application
On the second EC2 instance, check if your web server is running:

 Go to the public IP of your second EC2 instance in a web browser (http://<public-ip>).

 If everything is set up correctly, you should see your web application running!



Optional Enhancements
Scaling:
You can also set up auto-scaling groups for your EC2 instances and configure an Application Load Balancer (ALB) to distribute traffic among them. If you want to explore this next, I can guide you on configuring the load balancing.
Networking & Security:
Make sure your EC2 instances have public IPs and that security groups are configured to allow inbound traffic on port 80 (HTTP).


You might also want to set up a VPC with private and public subnets for better security.



Summary of Next Steps:
Create and build a Docker image (for your app).


Push it to ECR.


Pull the image on a different EC2 instance.


Run the Docker container on the second EC2 instance.


Verify your app by accessing it via the second EC2’s public IP.


Let me know if you need more details on any of these steps or if you'd like to proceed with any of the optional enhancements!

The lab setup you are working on is almost complete, but here’s a checklist to ensure every task is done correctly and to fully finish the lab:
Lab Checklist:
Create EC2 Instances:


You've already launched EC2 instances with proper IAM roles and security groups.


Ensure both EC2s are running Ubuntu 22.04 and have internet access.


Install AWS CLI and Docker on EC2:


You have successfully installed the AWS CLI and Docker on the EC2 instances. If not, make sure to run these installation steps on both EC2s.


# Install AWS CLI
sudo apt update && sudo apt install awscli -y

# Install Docker
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker


Create ECR Repository:


You have successfully created the ECR repository named my-ecr-website via the AWS CLI.


Make sure the repository is created in the correct region by checking the AWS Console or by running this:


aws ecr describe-repositories --region <your-region>


Dockerfile Creation:


Create the Dockerfile to build the image, including copying the static files or web application code to it.


Example Dockerfile for a simple web application:

 FROM nginx:latest

COPY ./website /usr/share/nginx/html

EXPOSE 80


Make sure you have a website folder containing static files (like index.html, style.css, etc.).


Build Docker Image:


Build the Docker image on the first EC2 instance:


docker build -t my-ecr-website .


Tag and Push the Image to ECR:


Tag the Docker image to match your ECR repository:


docker tag my-ecr-website:latest $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest


Push the image to the ECR repository:


docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest


Login to ECR from Second EC2 Instance:


On the second EC2 instance, authenticate Docker to AWS ECR:


aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com


Pull Docker Image from ECR:


Pull the Docker image from ECR on the second EC2 instance:


docker pull $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest


Run Docker Container on Second EC2:


Run the Docker container with the pulled image:


docker run -d -p 80:80 $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-ecr-website:latest


Verify the Application:


Open the public IP address of the second EC2 instance in a browser (http://<public-ip>).


You should see the web application running.



Optional Steps (To enhance your setup further)
Setting Up Auto-Scaling for EC2 Instances:


Create an Auto Scaling Group to automatically scale EC2 instances based on load.


Integrate it with an Application Load Balancer (ALB) to distribute traffic.


Advanced Networking:


You could configure the EC2 instances within a VPC with public and private subnets for more control over network traffic and security.


Add NAT Gateway and private subnets for secure communication between EC2 instances.


Set Up Security:


Modify the security groups to ensure only necessary ports (like HTTP 80 and SSH 22) are open.


Add a WAF (Web Application Firewall) if needed to protect your app from external attacks.


Set Up Monitoring:


You can configure CloudWatch Logs to monitor logs and performance metrics for your application.


Set up CloudWatch Alarms for critical issues like high CPU usage.


