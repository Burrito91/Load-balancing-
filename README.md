[Manual_Deployment_Guide for load balanced auto scaling web app.md](https://github.com/user-attachments/files/20844491/Manual_Deployment_Guide.for.load.balanced.auto.scaling.web.app.md)# Load-balancing-auto scaling web app
#This project provisions a load-balanced, auto-scaling web application on AWS. When the EC2 instances hit 100% CPU usage, the Auto Scaling Group automatically scales from 2 to 4 instances.


## 🎥 Video Walkthrough

📺 [Watch on YouTube](https://www.youtube.com/watch?v=Pi6RIb9rLFg&t=178s)



# 🧱 Project Architecture

[load balanced auto scaling web app diagram.pdf](https://github.com/user-attachments/files/20843834/load.balanced.auto.scaling.web.app.diagram.pdf)

**Key Components:**
- EC2 Launch Template  
- Auto Scaling Group (Min: 2, Max: 4 instances)  
- Application Load Balancer (ALB)  
- Target Group (attached to the ALB)  
- Amazon Linux 2 AMI  
- Manual stress test (no CloudWatch alarm configured)


 ## Full Project steps
 In this project, I created a basic but powerful AWS infrastructure that hosts a simple web app using EC2 instances. The system is designed to automatically scale out when CPU usage gets high, I used AWS services like Launch Templates, an Application Load Balancer (ALB), Auto Scaling Groups (ASG), and a little stress testing to simulate real-world traffic.


🛠️ Step 1: Created a Launch Template (No Key Pair Used)
The first thing I did was create a Launch Template — this is like a recipe that tells AWS how to launch EC2 instances automatically.

I picked Amazon Linux 2 as the base OS.

Chose t2.micro (free-tier eligible) as the instance type.

I did not select a key pair (so I wouldn’t be able to SSH, but that was okay for this project).

I created and attached a Security Group that allows:

Port 80 (for web traffic)

Port 22 (just in case I wanted to use EC2 Instance Connect)

The Launch Template helps the Auto Scaling Group know what kind of EC2 instance to launch, and with what settings.

🌐 Step 2: Added User Data to Install Apache Web Server
While creating the Launch Template, I also added a User Data script. This is a short Bash script that automatically installs and starts Apache whenever a new EC2 instance is launched.

bash
Copy
Edit
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
This sets up a basic website that shows which EC2 instance is handling the request (based on its hostname).

🎯 Step 3: Created a Target Group
Next, I created a Target Group, which is a collection of EC2 instances that sit behind the Load Balancer.

I chose Target Type: instance

Used HTTP protocol on port 80

Left the default health check settings (it checks / on port 80)

The Target Group is what the Load Balancer uses to route traffic to the right instances.

☁️ Step 4: Set Up an Application Load Balancer (ALB)
Then I created an Application Load Balancer — this makes sure that traffic is evenly distributed between all healthy instances.

I made it internet-facing so it’s accessible to users over the web.

I selected 2 public subnets in different Availability Zones (for high availability).

I created a listener on port 80, which listens for incoming web requests.

I connected it to the Target Group I made earlier.

Now, the ALB will direct traffic to the instances managed by the Auto Scaling Group.

🔁 Step 5: Created an Auto Scaling Group (ASG)
Now that the Launch Template, Target Group, and ALB were ready, I created an Auto Scaling Group.

I linked it to the Launch Template

I selected the same 2 Availability Zones as the ALB

I attached the Target Group and ALB to the ASG

I configured the scaling settings:

Minimum instances: 2

Desired instances: 2

Maximum instances: 4

This means the ASG would always keep 2 instances running and could scale up to 4 if needed — automatically.

🔥 Step 6: Simulated High CPU Usage (Without SSH Key)
Since I didn’t use a key pair, I couldn’t SSH from my terminal — but I was able to connect using EC2 Instance Connect, a browser-based terminal in the AWS Console.

Once I connected, I installed a tool called stress to simulate high CPU usage:

bash
Copy
Edit
# Enable EPEL (Extra Packages for Enterprise Linux)
sudo amazon-linux-extras enable epel

# Install EPEL and the stress tool
sudo yum install -y epel-release
sudo yum install -y stress

# Find out how many CPU cores the instance has
CPU_CORES=$(nproc)

# Use all cores to simulate CPU overload
stress --cpu "$CPU_CORES"
This caused the instance to hit 100% CPU, and after a few minutes, the Auto Scaling Group automatically launched more instances to handle the load.

📈 Step 7: Watched It Scale Automatically
After running the stress test:

I saw my EC2 instance count go from 2 → 3 → 4.

All new instances automatically registered with the Target Group.

The ALB began spreading traffic between them without me touching anything.

## 🧠 What I Learned
How to launch and auto-scale EC2 instances without SSH or key pairs

How to use Launch Templates to standardize EC2 creation

How to set up a User Data script to install a web server on boot

How to configure Application Load Balancer and Target Groups

How to simulate high CPU usage using the stress tool

That AWS Auto Scaling works out of the box with minimal configuration


## Deployment method
## 🚀 How to Deploy This Project on AWS

### 1. Create a Launch Template
- Go to *EC2 > Launch Templates*.
- Click *Create launch template*.
- Fill in:
  - Name (e.g. web-app-template)
  - Amazon Linux 2 AMI
  - t2.micro instance type
  - User data script to install Apache and the sample web page:
    
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Welcome to my Load Balanced Auto Scaling Web App!</h1>" > /var/www/html/index.html
    

---

### 2. Create an Auto Scaling Group
- Go to *EC2 > Auto Scaling > Auto Scaling Groups*.
- Click *Create Auto Scaling group*.
- Attach the *Launch Template* you created.
- Set:
  - *Minimum capacity:* 2  
  - *Desired capacity:* 2  
  - *Maximum capacity:* 4
- Choose *2 subnets* in different Availability Zones (AZs).
- Enable *Application Load Balancer* in the next step (see below).
- Set *Scaling policy*:
  - Target tracking policy: *Average CPU utilization = 80%*.

---

### 3. Create an Application Load Balancer (ALB)
- Go to *EC2 > Load Balancers > Create Load Balancer > Application Load Balancer*.
- Name it (e.g., web-alb)
- Scheme: *Internet-facing*, IP type: *IPv4*
- Choose *at least 2 subnets* in different AZs.
- Create a new *security group* that allows HTTP (port 80).
- Create a new *Target Group* of type *EC2 instances*:
  - Protocol: HTTP, Port: 80
- Listener: Add a rule to forward traffic to this target group.

---

### 4. Attach ASG to the Target Group
- Back in the Auto Scaling Group setup:
  - Select the *Target Group* you just created to register instances automatically.

---

### 5. Run CPU Stress Script (SSH into One EC2 Instance)
- Once your instances are running and reachable via the ALB, SSH into one instance:
  ```bash
  ssh -i your-key.pem ec2-user@<Public-IP>
Se

### 6. Run the cpu stress tool to simulate high Load 
sudo amazon-linux-extras enable epel
sudo yum install -y epel-release
sudo yum install -y stress
stress --cpu $(nproc)



## ✅ Conclusion
This project demonstrates how to build a simple yet scalable and highly available web application on AWS using core services like Launch Templates, Auto Scaling Groups, and Application Load Balancers,By using a startup script to automatically install a web server and a stress tool to simulate high CPU usage, I was able to observe real-time scaling behavior in response to demand.

What I learned is that with just a few well-configured AWS services, you can create an infrastructure that heals itself, scales automatically, and distributes traffic intelligently — without writing a single line of backend code or managing physical servers.


