Automated PHP Web Application Deployment Using Jenkins, GitHub and AWS EC2

1. Project Overview

This project demonstrates Continuous Integration and Continuous
Deployment (CI/CD) for a simple PHP web application.

Whenever code is pushed to the GitHub repository, Jenkins detects the
change, retrieves the latest source code, and deploys the application to
an Apache web server running on an AWS EC2 Ubuntu instance.

Workflow

```text
Developer (VS Code)
       |
       v
     GitHub
       |
       | Push event / webhook
       v
     Jenkins
       |
       | Freestyle project
       | Clone latest source code
       | Run deployment shell command
       v
  Apache Web Server
       |
       v
   AWS EC2 Instance
       |
       v
 Updated PHP Website
```

2. Technologies Used

• PHP and HTML — web application
• Git and GitHub — source code management
• Jenkins — CI/CD automation
• AWS EC2 — cloud virtual machine
• Ubuntu 24.04 LTS — operating system
• Apache — web server
• Java 21 — Jenkins runtime

3. Project Structure

```text
jenkins-deployment/
├── index.php
└── README.md
```

4. Create the PHP Application

Create a file named index.php and add:

```php
<?php
$message = "Welcome to my CI/CD project!";
$version = "Version 1.0";
?>

<!DOCTYPE html>
<html>
<head>
    <title>Jenkins CI/CD Demo</title>
    <style>
        body {
            background-color: #eaf2f8;
            font-family: Arial, sans-serif;
            text-align: center;
            padding-top: 100px;
        }
        .container {
            background: white;
            padding: 40px;
            margin: auto;
            width: 450px;
            max-width: 85%;
            border-radius: 12px;
        }
        h1 { color: #2874a6; }
    </style>
</head>
<body>
    <div class="container">
        <h1><?php echo htmlspecialchars($message); ?></h1>
        <h2>Automated Deployment Using Jenkins</h2>
        <p><?php echo htmlspecialchars($version); ?></p>
        <p>Deployed on AWS EC2.</p>
    </div>
</body>
</html>
```

5. Upload the Project to GitHub

1. Create a new GitHub repository named jenkins-deployment.
2. Open a terminal in the project directory.
3. Run the following commands. Replace YOUR_USERNAME with your GitHub
username.

```bash
git init
git add .
git commit -m "Initial PHP application"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/jenkins-deployment.git
git push -u origin main
```

If the Git remote already exists, update it instead of adding it again:

```bash
git remote set-url origin https://github.com/YOUR_USERNAME/jenkins-deployment.git
```

6. Launch an AWS EC2 Instance

1. Sign in to the AWS Management Console and open EC2.
2. Launch an Ubuntu Server 24.04 LTS instance.
3. Choose an instance type suitable for Jenkins and your workload. A
small instance may have limited memory; select an appropriately
sized instance for reliable operation.
4. Create or select an EC2 key pair.
5. Configure the instance security group.

Suggested inbound rules:

Service     Protocol/Port Source

────────

SSH                TCP 22 Your IP address
HTTP               TCP 80 0.0.0.0/0
Jenkins          TCP 8080 Your IP address or trusted network

Do not expose the Jenkins administration port or SSH to the entire
internet. For webhook delivery, configure a secure, appropriately
restricted route from GitHub to Jenkins. Prefer HTTPS and a reverse
proxy for a production deployment.

Launch the instance and wait until its state is Running.

7. Connect to EC2

1. Select the instance in the EC2 console.
2. Choose Connect.
3. Use EC2 Instance Connect or SSH with your private key.

Update Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

8. Install Java and Jenkins

Install Java 21:

```bash
sudo apt install fontconfig openjdk-21-jre -y
java -version
```

Add the official Jenkins LTS package repository and install Jenkins:

```bash
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
```

Enable and start Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Retrieve the initial administrator password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open Jenkins in a browser:

```text
http://EC2_PUBLIC_IP:8080
```

Replace EC2_PUBLIC_IP with the current public IPv4 address of your
instance. Enter the initial password, install the suggested plugins, and
create an administrator account.

9. Install Apache and PHP

Install Apache and PHP support:

```bash
sudo apt update
sudo apt install apache2 php libapache2-mod-php -y
```

Enable and start Apache:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2
```

Check PHP:

```bash
php -v
```

Check which Apache PHP module is available:

```bash
ls /etc/apache2/mods-available/php*
```

If a module is listed (for example, php8.3.conf), enable the matching
module. Use the version actually shown on your server:

```bash
sudo a2enmod php8.3
sudo systemctl restart apache2
```

Test Apache by opening:

```text
http://EC2_PUBLIC_IP
```

The Apache default page should appear before application deployment.

PHP verification

Create a temporary test file:

```bash
echo '<?php echo "PHP is working"; ?>' | sudo tee /var/www/html/php-test.php
```

Open http://EC2_PUBLIC_IP/php-test.php. The browser should display
PHP is working, not the PHP source code.

Remove the test file after verification:

```bash
sudo rm /var/www/html/php-test.php
```

10. Configure Deployment Permissions

Add Jenkins to the Apache web-server group:

```bash
sudo usermod -aG www-data jenkins
```

Set the web directory group and permissions:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 775 /var/www/html
sudo chmod g+s /var/www/html
```

Restart Jenkins so it starts with the updated group membership:

```bash
sudo systemctl restart jenkins
```

Verify the Jenkins account belongs to the www-data group:

```bash
id jenkins
```

The output should include www-data.

> Security note: These permissions are suitable for a basic classroom
> demonstration. For production, use a dedicated deployment directory
> and narrowly scoped permissions rather than broad write access to the
> web root.

11. Install Jenkins Plugins

In Jenkins:

1. Open Manage Jenkins → Plugins.
2. Ensure the Git plugin is installed.
3. Install the GitHub plugin if you plan to use GitHub webhook
integration.
4. Restart Jenkins if requested.

Also ensure Git is installed on the EC2 host:

```bash
sudo apt install git -y
git --version
```

12. Create a Jenkins Freestyle Project

1. Open the Jenkins dashboard.
2. Select New Item.
3. Enter the name PHP-Automated-Deployment.
4. Select Freestyle project.
5. Click OK.

Configure source code management

1. Open the job’s Configure page.
2. Under Source Code Management, select Git.
3. Enter the repository URL:

```text
https://github.com/YOUR_USERNAME/jenkins-deployment.git
```

4. Set the branch specifier to:

```text
*/main
```

For a public repository, credentials are normally not needed. For a
private repository, add an appropriate GitHub credential in Jenkins and
select it here.

Configure the build step

Under Build Steps, select Add build step → Execute shell and
enter:

```bash
set -e

echo "Starting deployment..."
echo "Copying application files to Apache web root..."

cp -r ./* /var/www/html/

echo "Deployment completed successfully."
```

Save the job.

The command copies the checked-out project files into the Apache
document root. If your repository contains files that should not be
published (for example, documentation or configuration files), use a
more selective deployment command.

13. Test the First Deployment

1. Open the Jenkins job.
2. Click Build Now.
3. Select the new build number in Build History.
4. Open Console Output.
5. Confirm that the log ends with:

```text
Deployment completed successfully.
Finished: SUCCESS
```

Open the application:

```text
http://EC2_PUBLIC_IP
```

The PHP application should be displayed.

14. Configure GitHub Webhook for Automatic Builds

A webhook notifies Jenkins when a push occurs. The GitHub plugin must be
installed and Jenkins must be reachable securely from GitHub.

Configure the Jenkins job

1. Open PHP-Automated-Deployment.
2. Click Configure.
3. Under Build Triggers, enable GitHub hook trigger for GITScm
polling.
4. Save the job.

Configure the GitHub repository

1. Open the GitHub repository.
2. Select Settings → Webhooks → Add webhook.
3. Set the Payload URL to:

```text
https://YOUR_JENKINS_HOST/github-webhook/
```

For a temporary classroom setup only, if Jenkins is reachable over HTTP,
the URL may use:

```text
http://EC2_PUBLIC_IP:8080/github-webhook/
```

4. Set Content type to application/json.
5. Select Just the push event.
6. Click Add webhook.

Check the webhook’s recent deliveries in GitHub if the job does not
start. A successful delivery means GitHub reached the endpoint; also
verify that the Jenkins GitHub plugin and job trigger are configured
correctly.

Security: Do not leave Jenkins administration openly accessible on
port 8080. Use HTTPS, restrict access, and configure network rules
carefully. A production deployment should use a secure reverse proxy and
appropriate authentication.

Alternative: Poll SCM

If a webhook cannot be used, open the Jenkins job configuration and
enable Poll SCM. Enter:

```text
H/2 * * * *
```

This checks for source-code changes approximately every two minutes.
Poll SCM is an alternative to webhook triggering; it is not required
when the webhook is working.

15. Demonstrate Automatic Deployment by Changing the Code

Open index.php on your local computer and change:

```php
$message = "Welcome to my CI/CD project!";
$version = "Version 1.0";
```

to:

```php
$message = "Welcome to my Updated CI/CD project!";
$version = "Version 2.0";
```

Save the file.

Push the changes:

```bash
git add .
git commit -m "Updated website to version 2.0"
git push origin main
```

Verify the Jenkins build

1. Open the Jenkins dashboard.
2. Open PHP-Automated-Deployment.
3. Check Build History.
4. A new build should start after the webhook notification is received
(or after the next SCM polling interval if Poll SCM is being used).
5. Open the new build and select Console Output.
6. Confirm the deployment completed and the build result is
SUCCESS.
7. Open the build’s Changes page to see the commit message and
changed files, when available.

Verify the updated website

Refresh:

```text
http://EC2_PUBLIC_IP
```

The page should now show:

• Welcome to my Updated CI/CD project!
• Version 2.0

This demonstrates that a GitHub push triggered Jenkins and the updated
PHP application was deployed to EC2.

16. Troubleshooting

────────

Problem                             What to check

────────

Jenkins page does not open          Jenkins service status, EC2 public
IP, and inbound TCP 8080 rule

Website does not open               Apache service status and inbound
TCP 80 rule

PHP source code appears in browser  Install php and
libapache2-mod-php, enable the
matching Apache PHP module, and
restart Apache

Jenkins cannot clone repository     Repository URL, branch name, Git
installation, and credentials for
private repositories

Jenkins build reports permission    Confirm id jenkins includes
denied                              www-data; restart Jenkins after
changing group membership

Webhook does not trigger a build    Check GitHub webhook delivery,
Jenkins GitHub plugin, job trigger,
and network access

Website does not update             Check the Jenkins console log,
deployment destination, and browser
cache

Jenkins build fails on shell        Review the complete Console Output command                             and correct the failing command

Useful commands:

```bash
sudo systemctl status jenkins
sudo systemctl status apache2
sudo systemctl restart jenkins
sudo systemctl restart apache2
php -v
id jenkins
ls -la /var/www/html/
```

17. Practical Demonstration Checklist

────────

```
                       No. Demonstration         Expected result
```

────────

```
                         1 Show GitHub           PHP source code is
                           repository            present

                         2 Show AWS EC2 instance Instance is running

                         3 Show Jenkins          Jenkins is accessible
                           dashboard             

                         4 Show Freestyle        Git repository and
                           project configuration deployment step are
                                                 configured

                         5 Run the first Jenkins Build is successful
                           build                 

                         6 Open the EC2 website  Version 1.0 is
                                                 displayed

                         7 Modify `index.php`    New commit appears in
                           and push to GitHub    GitHub

                         8 Show Jenkins Build    A new build is
                           History               triggered

                         9 Show Console Output   Deployment succeeds
                           and Changes           and commit is visible

                        10 Refresh the EC2       Version 2.0 is
                           website               displayed
```

────────

18. Result

Successfully developed and deployed a PHP web application using GitHub,
Jenkins Freestyle Project, Apache Web Server, and AWS EC2. Automated
deployment was configured so that changes pushed to the GitHub
repository are detected by Jenkins, retrieved from the repository, and
deployed to the Apache web directory on the EC2 instance.

19. Important Notes

• Replace YOUR_USERNAME, EC2_PUBLIC_IP, and YOUR_JENKINS_HOST
with your actual values.
• The EC2 public IP can change when an instance is stopped and started
unless an Elastic IP is assigned.
• Do not upload AWS access keys, private SSH keys, Jenkins passwords,
or other secrets to GitHub.
• Do not commit .pem files, .env files containing secrets, or
Jenkins credential files.
• The README.md documents the project; the deployable application is
index.php.
• Stop or terminate cloud resources when they are no longer needed to
avoid unexpected AWS charges.
----------------------------------------------------------------------------

Docker image creation

<!DOCTYPE html>
<html>
<head>
    <title>My Docker Web Application</title>
    <style>
        body {
            font-family: Arial;
            background-color: lightblue;
            text-align: center;
            padding: 50px;
        }

        h1 {
            color: darkblue;
        }

        p {
            font-size: 20px;
        }
    </style>
</head>
<body>

    <h1>Welcome to My Docker Project!</h1>

    <p>This PHP application is running inside a Docker container.</p>

    <p>Docker makes application deployment easy.</p>

    <?php
        echo "<h2>PHP is working successfully!</h2>";
    ?>

</body>
</html>

Dockerfile:
FROM php:8.2-apache

COPY index.php /var/www/html/index.php

EXPOSE 80

CMD ["apache2-foreground"]

command:
docker build -t my-web-app .


Run the docker:
docker run -d -p 8080:80 --name my-container my-web-app


cd C:\Docker-Project

dir

docker --version

docker info

docker build -t my-web-app .

docker images

docker run -d -p 8080:80 --name my-container my-web-app

docker ps
----------------------------------------------------------------------------

Terraform :
Configration code:

# Configure Terraform and AWS Provider

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

# Configure AWS Region

provider "aws" {
  region = "ap-south-1"
}

# Find Latest Amazon Linux AMI

data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name = "name"

    values = ["al2023-ami-2023*-x86_64"]
  }

  filter {
    name = "architecture"

    values = ["x86_64"]
  }
}

# Create EC2 Instance

resource "aws_instance" "my_server" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = "t3.micro"

  tags = {
    Name = "Terraform-EC2-Instance"
  }
}

# Display Public IP Address

output "instance_public_ip" {
  value = aws_instance.my_server.public_ip
}

# Display Instance ID

output "instance_id" {
  value = aws_instance.my_server.id
}

terraform init
terraform validate
terraform plan
terraform apply
---------------------------------------------------------------------------