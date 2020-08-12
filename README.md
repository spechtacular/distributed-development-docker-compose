This environment has been tested on Macs running Sierra.
Windows users, contact Ted, lets walk through the process together.

Installed git, docker, and postman using brew.

You will need an account on https://www.docker.com/, this will allow you to pull images from their community registry.
SQL Server and Ubuntu Docker images are pulled from the registry.
--- Marcg 7, 2018
1. Testing gitlab backup

--- March 1, 2018
1. Now using AWS ECR as our docker registry.
2. Added .env file containing URLs and Version IDs of the docker images in AWS ECR.
3. An AWS ECR repository URL was added to each projects Dockerfile FROM command.
3. Used ansible to provision AWS, docker, and git on these Linux VMs only:
   * 10.81.1.228
   * 10.81.1.229
   * 10.81.1.232
4. AWS authentication is required before running docker-compose, copy and paste these scripts, depending on the platform:
   * Mac:  $(aws ecr get-login --no-include-email --region us-west-1)
   * Linux: sudo $(aws ecr get-login --no-include-email --region us-west-1) 
   * The AWS authentication token expires after 12 hours.

---
#### Git usage

1. git submodules added complexity (pain)
    * use the --recursive flag on the git clone command.
    * git push --recurse-submodules=on-demand
    * git submodule update --remote --merge
2. Using git submodules in the verity_api_docker.git project to integrate the services into a Verity API environment where gitlab-vhs-api-docker is an alias in the .ssh/config file
   https://git-scm.com/book/en/v2/Git-Tools-Submodules
   Add submodules to main module
   git submodule add ssh://gitlab/vhs-api-sqlserver.git
   cd verity_api_sqlserver
   git submodule init
   git submodule update
   Add another submodule
   git submodule add ssh://gitlab/vhs-api-nodeserver.git
   cd verity_api_webservice
   git submodule init
   git submodule update
   Add another submodule
   git submodule add ssh://gitlab/vhs-api-postman.git
   cd verity_api_integration_testing
   git submodule init
   git submodule update
   Clone the project and all submodules in one command
   git clone --recurse-submodules ssh://gitlab/vhs-api-docker.git
   Pull master branch updates to submodules
   cd vhs-api-docker
   git submodule update --remote vhs-api-postman
   git submodule update --remote vhs-api-sqlserver
   git submodule update --remote vhs-api-nodeserver
   Then:
   commit and push the parent module 


---
#### Docker usage

1. docker -v
  * Docker version 17.09.0-ce, build afdb6d4
  * docker-compose command line usage:
  * Build and start the docker containers:
2. docker-compose up --build
  * This line in stdout log stream indicates the docker containers are ready for testing:
  * "sqlserver_1  | ======= MSSQL CONFIG COMPLETE ======="
  * Check the running containers from a terminal:
3. docker ps
  * CONTAINER ID        IMAGE                                              COMMAND                  CREATED             STATUS                       PORTS                    NAMES
  * 98f94e7077cb        verity_consolidated_daily_dashboard/node:v1        "npm start"              About an hour ago   Up About an hour             0.0.0.0:3000->3000/tcp   veritymetricsdocker_web_1
  * a0f1edd6ee49        verity_consolidated_daily_dashboard/sqlserver:v1   "./entrypoint.sh '..."   About an hour ago   Up About an hour (healthy)   0.0.0.0:1433->1433/tcp   veritymetricsdocker_sqlserver
  * Check the docker images from the command line.
    * docker inspect verity_consolidated_daily_dashboard/node:v1
4. docker images
  * REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
  * verity_consolidated_daily_dashboard/node        v1                  438a946045b3        About an hour ago   521MB
  * verity_consolidated_daily_dashboard/sqlserver   v1                  7d474d75c965        21 hours ago        1.33GB
  * ubuntu                                          16.04               00fd29ccc6f1        3 weeks ago         111MB
  * microsoft/mssql-server-linux                    latest              b0bb2e68f091        6 weeks ago         1.33GB
5. docker-compose down
  * shutdown project containers:
6. docker rmi 438a946045b3
  * remove unused project images from the command line.
7. cleanup docker resources
  * delete volumes
    * docker volume rm $(docker volume ls -qf dangling=true)
  * delete networks
    * docker network prune
  * remove docker images
    * docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
  * remove exited containers
    * docker rm $(docker ps -qa --no-trunc --filter "status=exited")
  * remove untagged images
    * docker rmi -f $(docker images | grep "<none>" | awk "{print \$3}")
8. manage docker containers
  * list containers
    * docker ps
  * stop container
    * docker stop <containerid>
  * remove container
    * docker rm <containerid>
   

---
#### The nodejs web application can be accessed at: localhost:3000

JSON Web Token authentication is used as a stateless authentication mechanism.
JWT Authorization header is used to control access to protected assets.

Routes implemented:

1. unprotected URLs:
  * post('/register', users.register);
  * post('/login', auth.login);

2. metrics routes that can be accessed only by authenticated metrics users:
  * get('/api/v0.1/metrics', metrics.getOne);

3. user routes that can be accessed only by authenticated & authorized admin users:
  * get('/api/v0.1/admin/users',  users.getOne);
  * delete('/api/v0.1/admin/users/:id', users.delete);

---
#### Postman is used as an automated API test tool.
It is a native application with a GUI. (some older versions run in the Chrome browser)

1. Postman test files for the project are in the verity_metrics_docker/verity_consolidated_daily_dashboard/postman folder.
2. Import the Postman collection file verity_metrics_docker/verity_consolidated_daily_dashboard/postman/verity.postman_collection.json  into the Postman app.
  * Import->Import File
3. Import the Postman environment file verity_metrics_docker/verity_consolidated_daily_dashboard/postman/Verity.postman_environment.json into the Postman app.
  * Upper right Gear icon->Import->Choose Files
4. Run the test collection:
  * Runner->Choose a collection->verity, Environment->Verity, Start Run
5. Postman should run 23 tests, all of these tests should pass.

  * To run individual Postman tests after loading the collection and environment:
  * In the left panel->Collections, click on verity, this should list the tests in the order they are run.
  * Click on a test in the Collections panel, it will create a request tab in the Builder panel, click Send.
  * The accounts must be registered first, execute the requests in the same sequence they appear in the Collections Panel.
   
 
