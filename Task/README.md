# TravelMemory Mern Application Deployment


# 1. Backend Configuration
**a. Launch EC2 Instance**:
  Log in to your AWS Console and start a new EC2 instance (Ubuntu is recommended).
  Choose an instance type (t2.medium or higher for better performance).
  Configure security group: open ports 22 (SSH), 80 (HTTP), and 443 (HTTPS).
  Connect to the server via SSH.

**b. System Setup:**
  Update the system:
  sudo apt update && sudo apt upgrade -y
  Install Node.js and npm:

  curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
  sudo apt-get install -y nodejs
  Install Git

**c. Clone and Configure Backend:**

  Clone the repository:
  git clone https://github.com/UnpredictablePrashant/TravelMemory.git
  cd TravelMemory/backend

  npm install
  Create and update .env with database URI and port info
  
  text
  MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/dbname
  PORT=3000
  d. Set Up NGINX Reverse Proxy:

Install NGINX:

text
sudo apt install nginx -y
Configure NGINX for reverse proxy:

text
sudo nano /etc/nginx/sites-available/default
Add/replace with:

text
server {
    listen 80;
    server_name your_domain.com;
    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    location / {
        proxy_pass http://localhost:3000/; # Change if frontend on different port/instance
    }
}
Save and restart NGINX:

text
sudo systemctl restart nginx
e. Start Backend:

Use PM2 (process manager) for resilience:

text
npm install -g pm2
pm2 start index.js --name travel-memory-backend
pm2 startup
pm2 save
2. Frontend and Backend Connection
a. Configure Frontend:

Go to frontend directory:

text
cd ../frontend
Install dependencies:

text
npm install
Update urls.js to point to your backend API endpoint:

js
export const API_URL = 'http://your_domain.com/api/';
Replace your_domain.com with your actual domain or EC2 public IP.

b. Build the Frontend:

text
npm run build
c. Serve with NGINX (Optional):

Move build files to NGINX’s web directory:

text
sudo cp -r build/* /var/www/html/
Make sure NGINX serves static files correctly.

3. Scaling the Application
a. Launch Multiple EC2 Instances:

Repeat the above steps to create identical backend/frontend servers on multiple EC2 instances.

b. Configure AWS Elastic Load Balancer (ELB):

Go to EC2 console > Load Balancers > Create Load Balancer.

Choose Application Load Balancer.

Add your EC2 instances to the target group.

Configure health checks (path: /api/health if available).

Set listeners for HTTP (80) and HTTPS (443).

c. Update Security Groups:

Ensure all connected EC2 instances allow traffic from the ELB.

4. Domain Setup with Cloudflare
a. Register and Add Domain to Cloudflare:

Sign up at cloudflare.com and add your domain.

Update your domain registrar to use Cloudflare’s nameservers.

b. DNS Configuration:

CNAME Record:

Name: www

Target: Load balancer DNS endpoint (from AWS)

A Record:

Name: @

IPv4 address: Public IP of EC2 instance hosting the frontend (if static IP).

c. Security and SSL:

Enable "Flexible" or "Full" SSL in Cloudflare’s dashboard.

Redirect HTTP to HTTPS in Cloudflare settings.

5. Documentation and Diagram
a. Documentation Checklist:

For each step above, take relevant screenshots in AWS and Cloudflare dashboards.

Explain commands and config changes with details.

List common troubleshooting tips (e.g., port issues, NGINX errors).

Create and annotate a deployment architecture diagram in draw.io showing:

User → Cloudflare → AWS Load Balancer → Multiple EC2 Instances (frontend/backend) → MongoDB.

Deliverables and Submission
Make sure the application works at www.yourdomain.com.

Compile all screenshots and stepwise documentation in a PDF/Markdown file.

Export the draw.io diagram as a PNG/JPEG and attach it.

Evaluation Tips
Double-check security group and NGINX rules (allow only necessary ports).

Use environment variables for credentials.

Keep your EC2 updated and Node processes managed (PM2).

Ensure health checks for ELB.

Validate cloudflare DNS propagation and SSL health.

Deploying this MERN application as per above steps will help meet all assignment objectives and best practices for a robust cloud deployment.

for step 4. Domain Setup with Cloudflare, we need buy a domain and setup the DNS record in cloudfare. it is bit lengthy process, is there any alternate solution to submit assignment to hit domain?

for step 4. Domain Setup with Cloudflare, we need buy a domain and setup the DNS record in cloudfare. it is bit lengthy process, is there any alternate solution to submit assignment to hit domain?
There are several alternatives to buying a domain and setting up DNS with Cloudflare when submitting your assignment:

Use the AWS Load Balancer DNS Name: After setting up the AWS Load Balancer for your application, AWS provides a unique DNS endpoint (like my-alb-1234567890.us-east-1.elb.amazonaws.com). You can share this DNS name as the public endpoint for your deployed application. This works for demonstration and assignment purposes, as anyone can access your site through this address without a custom domain.

Free or Temporary Domain Providers: Some services (such as DuckDNS, Freenom, or Netlify for static sites) offer either free domains or subdomains, which you can point to your AWS instance or load balancer. These are useful for temporary projects and can be set up quickly for testing and demonstration.

Access via Public IP: You can also directly use the public IP of your EC2 instance for access (e.g., http://<ec2-public-ip>). For load-balanced deployments, you should use the AWS Load Balancer DNS name, since that handles multiple instances and failover.

Summary:
The quickest and simplest no-purchase solution is to use the AWS Load Balancer DNS endpoint as your application's "domain" for the assignment. This satisfies accessibility requirements, supports load balancing, and eliminates the need for a paid or free external domain setup.

If your assignment requires a "domain name" strictly, clarify whether the AWS-provided DNS name is acceptable—most instructors accept this for cloud deployment exercises.

Test via the private IP of the which target group, because in the task it's mentioned as launch multiple instances and assigned to load balancer.

Test via the private IP of the which target group, because in the task it's mentioned as launch multiple instances and assigned to load balancer.
You should test the /health endpoint on the private IP of each EC2 instance that is registered in your Load Balancer's target group.

Since the task is to launch multiple instances and register them with the Load Balancer, the health check evaluates each registered instance individually on its private IP and port.

Why Test Private IP of Each Target?
AWS Load Balancer sends health check requests to the private IP and port of each target instance.

Public IP or localhost may not reflect the real network path the load balancer uses.

Testing private IP verifies the actual health check path accessibility as the load balancer sees it.

How to Test?
For each instance:

Identify the private IP from AWS EC2 Console > Instances.

On any machine inside the same VPC (including the instance itself), run:

bash
curl -i http://<PRIVATE_IP>/health
Expect to get HTTP/1.1 200 OK and the response body.

If the health check succeeds on all instance private IPs but the Target Group still shows unhealthy:

Check security group rules to ensure the LB can access all these private IPs on port 80.

Double-check NGINX config on each instance to serve /health consistently.

Wait for AWS health check intervals to update the target status.

This testing step applies to each instance registered with the ALB Target Group for accurate multi-instance validation.




