# udemy-docker-complex

### Basic Architecture

![image](https://user-images.githubusercontent.com/9342308/72189583-3348ad00-33cb-11ea-8e97-6edd3b947911.png)

### Basic Architecture

![image](https://user-images.githubusercontent.com/9342308/72027322-88aa8000-324c-11ea-8fb6-9e8d8186ca71.png)

### Dev Environments

We need a separate dev container for each of these three items. So that when we make changes to on of them, only that single container gets rebuilt

![image](https://user-images.githubusercontent.com/9342308/72027407-e8a12680-324c-11ea-9f24-1ed1bcadcc77.png)

### Nginx Splits client (React Server) vs server (Express)

Note: Client and Server (on right) are considered "upstream"

![image](https://user-images.githubusercontent.com/9342308/72192957-8f183380-33d5-11ea-86dd-eb054c0aceda.png)

### Starting Production Build

NOTE: React Server is different, now separate Nginx that is simply serving static files

![image](https://user-images.githubusercontent.com/9342308/72203509-7857f800-343a-11ea-9acc-6049b8d7e6f7.png)

### Built .travis.yml that builds images and pushes them to Docker Hub

### Dockerrun.aws.json

-   At least one container must be "essential"
-   "hostname" is optional for both worker and nginx, as other containers don't need to contact those containers
-   docker-compose is better at implicit links/hostnames between containers. Must be defined in AWS ECS

```json
{
    "AWSEBDockerrunVersion": 2,
    "containerDefinitions": [
        {
            "name": "client",
            "image": "cburkins/multi-client",
            "hostname": "client",
            "essential": false
        },
        {
            "name": "server",
            "image": "cburkins/multi-server",
            "hostname": "api",
            "essential": false
        },
        {
            "name": "worker",
            "image": "cburkins/multi-worker",
            "hostname": "worker",
            "essential": false
        },
        {
            "name": "nginx",
            "image": "cburkins/multi-nginx",
            "hostname": "nginx",
            "essential": true,
            "portMappings": [
                {
                    "hostPort": 80,
                    "containerPort": 80
                }
            ],
            "links": ["client", "server"]
        }
    ]
}
```

### Creating Elastic Beanstalk Application

1. Create Application
1. Create Environment within Application
1. Pre-configured platform: Multi-container Docker
1. Application code: Sample Application

### Running Production Databases

Instructor recommends using outside data services (e.g. AWS ElastiCache or RDS) rather than your own container

1. More scalable than your own Redis within a container
1. Built-in logging and maintenance
1. Probably better data security than we can do
1. Automatic backups and rollbacks (RDS Relational Database Service)
1. Easier to migrate off of EB

NOTE: Next project in this course will setup production database in a docker container

![image](https://user-images.githubusercontent.com/9342308/72212545-1932cb80-34ac-11ea-9dc1-8de04122d8e3.png)

By default, this services can't talk to each other (i.e. Elastic Beanstalk can't talk with ElastiCache or RDS)

![image](https://user-images.githubusercontent.com/9342308/72212626-58ade780-34ad-11ea-95ba-6bf9c7d6eddf.png)

#### Creating PostgreSQL database instance and database via RDS

1. Log into AWS
1. Select RDS
1. Scroll down to "Create Database" section
1. Click on "Create Database" button

    ![image](https://user-images.githubusercontent.com/9342308/72212705-e4744380-34ae-11ea-96ec-a0716c457415.png)

1. Click on "PostgreSQL"
1. Under templates, click on "Free Tier" (uses t2.micro for instance, cheaper)
1. DB instance identifier: multi-docker-postgres
1. Master username: postgres
1. Master password: <b>[be sure to remember this]</b>
1. Under Connectivity: Make sure publicy-accessible is NOT checked
1. Under Additional Configuration: set "database name", otherwise AWS will simply create the database instance, and not an actual database (for this example, using "fibvalues")

#### Creating an AWS ElastiCache Cluster Instance (Redis)

1. Log into AWS, Select "ElastiCache" service
1. Click on "Redis" on the left
1. Click on "Create"
1. Name: multi-docker-redis
1. <b>Select Node Type</b> as "cache.t2.micro"
1. Number of Replicas: change from 2 to 0
1. Subnet group: Create New
1. Name: redis-group (not important)
1. VPC ID: Select your default VPC

#### Creating Security Group

1. In AWS, go to VPC or EC2, select "Security Groups" on left
1. Click on "Create Security Group"
1. Security Group Name: multi-docker
1. VPC: [select your default VPC]
1. Click Create
1. Go to Inbound Rules
1. Create
1. Custom TCP Rule, TCP, Ports 5432-6379, Source=thisSecurityGroup

Now apply this new security group (named "multi-docker") to all three instances (ElastiCache, RDS instance, and Elastic Beanstalk Instance)

For example....

![image](https://user-images.githubusercontent.com/9342308/72212919-353a6b00-34b4-11ea-851c-2a76b0d2556e.png)

#### Update ElasticBeanstalk with Environment Variables

<b>NOTE: AWS does not obfuscate these values, so your database password will be visibile to other's who have access to this screen in your AWS Account</b>

1. On AWS, go to Elastic Beanstalk
1. Open the correct environment
1. On left, click on "Configuration"
1. On "Software" category, click "Modify"
1. Scroll down to Environment Properties
    1. REDIS_HOST (this is "Primary Endpoint" from ElastiCache config screen, removing ":6379" at end )
    1. REDIS_PORT
    1. PGUSER
    1. PGPASSWORD
    1. PGHOST (this is "Endpoint" from RDS config screen)
    1. PGDATABASE
    1. PGPORT
