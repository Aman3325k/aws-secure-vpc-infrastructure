 # AWS Production-Grade Secure VPC Infrastructure

    A highly available, secure, and production-grade AWS VPC architecture spanning two Availability
  Zones. This project demonstrates the AWS-recommended best practice for hosting web applications by
  completely isolating backend application servers in private subnets, exposing only an Application
  Load Balancer (ALB) to the public internet, and using a Bastion Host (Jump Box) for secure
  administrative access.

    ## Architecture Diagram

    ```mermaid
    graph TD
        Internet[Internet / Users] --> IGW[Internet Gateway]
        IGW --> ALB[Application Load Balancer <br> Public Subnet]

        subgraph VPC [AWS VPC: 10.0.0.0/16]
            subgraph AZ_1 [Availability Zone 1: eu-north-1a]
                subgraph Public_Subnet_1 [Public Subnet 1]
                    ALB_1[ALB Node]
                    NAT_1[NAT Gateway]
                    Bastion[Bastion / Jump Host]
                end
                subgraph Private_Subnet_1 [Private Subnet 1]
                    EC2_1[App Instance 1 <br> Python Server on Port 8000]
                end
            end

            subgraph AZ_2 [Availability Zone 2: eu-north-1b]
                subgraph Public_Subnet_2 [Public Subnet 2]
                    ALB_2[ALB Node]
                end
                subgraph Private_Subnet_2 [Private Subnet 2]
                    EC2_2[App Instance 2 <br> Python Server on Port 8000]
                end
            end
        end

        ALB --> EC2_1
        ALB --> EC2_2
        EC2_1 --> NAT_1
        EC2_2 --> NAT_1
        NAT_1 --> IGW

  ## Key Infrastructure Features

  • Secure Networking (VPC): Custom VPC (10.0.0.0/16) divided into 2 Public Subnets and 2 Private
  Subnets across two Availability Zones for high availability.
  • Backend Isolation: Application servers are deployed strictly inside Private Subnets with no
  public IPs, making them invisible and inaccessible from the public internet.
  • Traffic Balancing (ALB): An Internet-Facing Application Load Balancer distributes user requests
  on Port 80 (HTTP) to the private instances on Port 8000.
  • Administrative Access (Bastion Host): A secure Jump Box launched in the public subnet allows SSH
  access into the private instances using key pair authentication.
  • Secure Internet Outgoing (NAT Gateway): A NAT Gateway allows the private EC2 instances to
  connect out to the internet to download updates and packages, while blocking unsolicited inbound
  connections.
  • Auto Recovery: An Auto Scaling Group (ASG) manages the private instances to ensure a desired
  capacity of 2 instances is always running.

  ## Deployment & Verification Steps

  ### 1. Connecting via the Bastion Host

  Since the application servers do not have public IPs, you must route your connection through the
  Bastion Host:

    # Secure local SSH key permissions
    chmod 400 your-key.pem

    # Copy your private key to the Bastion Host (required for the second jump)
    scp -i your-key.pem your-key.pem ubuntu@<BASTION-PUBLIC-IP>:/home/ubuntu/

    # SSH into the Bastion Host
    ssh -i your-key.pem ubuntu@<BASTION-PUBLIC-IP>

  ### 2. Accessing the Private Server

  Once logged inside the Bastion Host:

    # Secure the key file permissions on the Bastion
    chmod 400 your-key.pem

    # SSH from the Bastion into the private EC2 instance using its private IP
    ssh -i your-key.pem ubuntu@<PRIVATE-INSTANCE-IP>

  ### 3. Running the Application

  Create a simple landing page and start the built-in Python web server on Port 8000:

    echo "<h1>AWS Production-Grade Web Infrastructure Verified!</h1>" > index.html
    python3 -m http.server 8000

  ### 4. Testing the Architecture

  Copy the public DNS name of your Application Load Balancer from the AWS Console and paste it into
  your browser:

    http://<YOUR-ALB-DNS-NAME>

  The load balancer will receive the request on Port 80 and successfully forward it to the active
  Python server on Port 8000 in your private subnet.

  ## Skills Demonstrated

  • AWS Networking (VPC, Subnets, Route Tables, Internet Gateways, NAT Gateways)
  • Security Group configurations & administrative separation (Bastion Hosts)
  • High Availability architectures using Application Load Balancers & Target Groups
  • Command-line server management & secure network traversal (ssh, scp)

