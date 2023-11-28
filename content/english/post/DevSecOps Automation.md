+++
author = "M0D4S"
title = "Automating the implementation of a DevSecOps Environment and pipeline"
date = "2023-09-08"
description = "Automating the creation of an environment and pipeline for DevSecOps in a private cloud"
tags = ["devsecops", "automation", "terraform", "ansible", "jenkins", "sonarqube", "sca", "sast", "dast", "compliance", "openstack", "DevStack", "cloud", "security", "devops", "ci/cd", "pipeline", "zap", "owasp", "owasp-zap"]
+++

## Architecture of the solution ..
<!-- Import an image of the architecture -->
![Traefik Load Blancer with docker swarm](/images/portfolio/SecOps.png)

<p>
The image above presents the architecure of the solution. The solution is composed of :
<ul>
<li>A Jenkins server that is used for the CI/CD pipelines</li>
<li>A Jenkins agent that is used to build the code and run the security tests</li>
<li>A SonarQube server that is used to analyze the code</li>
</ul>

The phases of the Security testing are:
<ul>
<li>Build the code</li>
<li>Run the SCA (Software Composition Analysis) tests</li>
<li>Run the SAST (Static Application Security Testing) tests</li>
<li>Run the DAST (Dynamic Application Security Testing) tests</li>
<li>Run the compliance test of the instance hosting the application</li>
</ul>
</p>

<!--PS: in red, the project uses a previous project of mine made with django as a security and build subject you can change that yourself in the SCM Checkout of the code and the build in the zap pipeline-->

<p style="border: 2px solid red; padding: 10px;">
<strong style="color: red;">PS:</strong> This project uses a previous project of mine made with django as a security testing and build subject, you can change that yourself in the SCM Checkout of the code and the build in the zap pipeline.
</p>

To achieve this, we require terraform to allocate the resources in the cloud and ansible to configure the servers.

## How to implement it ?
<p>
First, we need to have a private cloud up and running, we can use openstack for that, both the DevStack and MiniStack versions enables the creation of a single node private cloud. Next, we need to install terraform and ansible along with a few other tools on our machine. We need to create terraform scripts to allocate the resources in the cloud and ansible playbooks to configure the servers. <br>
Thankfully, I did all of that and I created a repository that contains all the scripts and playbooks that we need to implement the solution. The repository is available on my github account, you can find it <a href="https://github.com/Yassine-Rejeb/SecOps_OpenStack" target="_blank">here</a>.
</p>

## How to deploy it ?
<p>
To deploy the solution, we need to follow these steps:
<ol>
<li>Clone the repository:
<pre><code>git clone https://github.com/Yassine-Rejeb/SecOps_OpenStack.git </code></pre>
</li>
<li>Rename the cloned repository to "SecOps":
<pre><code>mv SecOps_OpenStack SecOps</code></pre>
</li>
<li>Go to the SecOps directory:
<pre><code>cd SecOps</code></pre>
<li> Download the OS image:
<pre><code>./setup/getCentOSCloud.sh</code></pre>
</li>
<li>Change the IP @ in the file setup/openrc with the IP of an interface that has access to internet. The line is: export OS_AUTH_URL=http://{{IP@}}/identity
</li>
</li>
<li>Start with running the prepare_infra.sh script:
<pre><code>./prepare_infra.sh</code></pre>
</li>
<li>Follow the instructions on the screen to create the infrastructure, This may take a while. And If you do not have enough resources on your machine, you may need to restart the script a few times. <br>
Once the infrastructure is created, you can run the configure.sh script to configure the servers:
<pre><code>./configure.sh</code></pre>
</li>
<li>Follow the instructions on the screen VERY CAREFULLY to configure the servers. <br>
Once the servers are configured, you can open your browser and go to the Jenkins server IP address to access the Jenkins server (Creadentials are M0D4S/M0D4S). <br>
When you authenticate, you will find some pipelines that are already configured and ready to be executed. As discussed earlier, these pipelines are using the django project that I created as a subject for the security testing. You can change that yourself in the SCM Checkout of the code (in each pipeline) and the build in the zap pipeline.
</li>
</ol>
</p>

### Here are some screenshots of the solution:
<p>
Running the prepare_infra.sh script:
</p>

![prepare_infra.sh](/images/portfolio/SecOps/prepare_infra.png)

<p>
Running the configure.sh script:
</p>

![configure.sh](/images/portfolio/SecOps/configure.png)

<p>
The Jenkins pipelines:
</p>

![Jenkins server](/images/portfolio/SecOps/jenkins.png)

<p>
The SonarQube server:
</p>

![SonarQube server](/images/portfolio/SecOps/sonar.png)

## How to destroy it ?
<p>
To destroy the infrastructure, you can go to the SecOps/terraform directory and run:
<pre><code>source ../setup/openrc</code></pre>
<pre><code>terraform destroy</code></pre>
</p>

## Conclusion
<p>
This project was a great experience for me, I learned a lot of things about DevSecOps and I had a lot of fun doing it. <br>
I hope that you enjoyed reading this article and that you learned something new. <br>
If you have any questions, please feel free to contact me.
If you liked this Article/Project, please share it with your friends and colleagues and star the repository on github.
Thank you for reading.
</p>