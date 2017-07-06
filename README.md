# NeoLoad Web deployment guide

NeoLoad Web deployment is available via docker compose files, to run NeoLoad Web on premise (dockerise NeoLoad Web).

This deployment is based on Docker and docker-compose - **only available for Linux**.

## NeoLoad Web architecture

NeoLoad Web is, in this docker context, decomposed in 3 main components:

```
Database - a MongoDB database (3.2.x version)

   ^
   |
   |
   +
Backend  - the backend services - communicating with the database

   ^
   |
   |
   +
Frontend - the web interface - communicating with the backend
```

## Deployment types 

- **All in one deployment**

The simplest way to deploy NeoLoad Web, all 3 components will be deployed for you.

- **External MongoDB deployment**

MongoDB is already available. NeoLoad Web will be deployed using your MongoDB. 
Only the front-end and the backend are deployed, some configuration is required to use your MongoDB.

- **All in one SSL deployment**

Secured version of the "All in one" deployment (add NGINX as a facade to secure HTTP traffic).

- **External MongoDB SSL deployment**

Secured version of the "External MongoDB" deployment

## Prerequisites

#### 1. This deployment is only intented to be used for Linux OS.

#### 2. Hardware requirements

This topic provides the hardware requirements sized for:

- 5 simultaneous running benches with:
  - 4 tests having 1000 requests/s for 30 transactions having 10000 VUs.
  - 1 test having 4000 requests/s for 40 transactions having 20000 VUs.

Hardware requirements depends of the deployment types you have chosen.

**2.1. All in one deployments**

- CPU: recent AMD or Intel processors with 8 cores at 3,6 Ghz or faster
- Minimum 8GB of physical memory, 16 GB recommended
- HDD: SSD disk, especially for MongoDB data directory (disk speed has an huge impact on the database performance)  

  Note: size of the disk depends on the quantity of **bench hours** you want to keen in NeoLoad Web.  
  
  Data consumption is about 200MB per hour.
  
  For example, if you want your NeoLoad Web to be able to keep 100 hours of benches without being required to remove old benches:
  > 100 hours x 200MB = 20GB  

**2.2. External MongoDB deployments**

2.2.1 Your external MongoDB hardware requirements

Requirements for a single MongoDB machine:

- CPU: recent AMD or Intel processors with 8 cores at 3,6 Ghz or faster
- Minimum 8GB of physical memory, 16 GB recommended
- HDD: SSD disk, especially for MongoDB data directory (disk speed has an huge impact on the database performance)  

2.2.2 NeoLoad Web deployment

- CPU: recent AMD or Intel processors with 2 cores at 2 Ghz or faster
- 4GB of physical memory
- HDD: no special requirements about the HDD type (less than 1 GB of free disk space is required)


#### 3. *Docker* and *docker-compose*

In order to deploy NeoLoad Web you need to make sure you are running at least [docker 1.12.x](https://docs.docker.com) and [docker-compose 1.7](https://docs.docker.com/compose). 

- *Docker* installation

    - for Ubuntu: [install docker](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
    - for Debian: [install docker](https://docs.docker.com/engine/installation/linux/docker-ce/debian/)
    - for Fedora: [install docker](https://docs.docker.com/engine/installation/linux/docker-ce/fedora/)
    - for CentOS: [install docker](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)

- *docker-compose* installation

    [install docker-compose](https://docs.docker.com/compose/install/)

For global installation instructions please refer to the official websites:

- https://docs.docker.com/installation
- https://docs.docker.com/compose

## All in one deployment

1. #### Configure

    1.1. get [docker-compose-all-in-one.yaml](http://TODO)
    
    1.2. edit the file according to your environment
    
    - MongoDB data directory (**required**)
    
      Create a directory for your MongoDB data.  
    
      Change **PATH_TO_YOUR_MONGODB_DIRECTORY** to the path of your data directory. 
      
      > This is a mapping allowing persistence of a volume when your docker containers are restarted. This way, the MongoDB data is not volatile anymore   
    
        ```
        mongo:
            ...
            volumes:
            - PATH_TO_YOUR_MONGODB_DIRECTORY:/data/db
            ...
        ```
    - Exposed backend ports (**optional**)
    
      3 ports are exposed with default values, change it if needed:
          
      - NeoLoad Controller REST port: ```8080```
      
      This is the port which will be used for communication between NeoLoad Controller and NeoLoad Web.
      
      ```
      nlweb-backend:
            ...
            ports:
                - 8080:1081
            ...
      ```
       
      - Public REST API port: ```8081```
      
      This is the [public REST API](https://www.neotys.com/documents/doc/nlweb/latest/en/html/#25050.htm) port which is exposed by default on the backend.
      
      ```
        nlweb-backend:
            ...
            ports:
                ...
                - 8081:1082
            ...
       ```
      
      - NeoLoad Web backend monitoring : ```8092```
      
      For monitoring purpose only, enable retrieval of information relative to the state of the backend services.
      
      ```
        nlweb-backend:
          ...
          ports:
              ...
              - 8092:9092
          ...
       ```
    
    - Exposed frontend ports (**optional**)
    
        2 ports are exposed with default values, change it to your needs if required:
              
        - NeoLoad Web interface: ```80```
          
        This is the port to be used to access the web interface.
          
        ```
        nlweb-frontend:
            ...
            ports:
                - 80:9090
            ...
        ```
        
        - NeoLoad Web frontend monitoring: ```81```
          
        For monitoring purpose only, enable to retrieve information relative to the state of the frontend services.
          
        ```
        nlweb-frontend:
            ...
            ports:
                - 81:9091
            ...
        ```

2. #### Run
    
    2.1. run the docker-compose file
    
    ```
        > docker-compose --file docker-compose-all-in-one.yaml up --remove-orphans
    ```

    2.2. Look at the logs to check if the deployment is going fine (**optional**)
    
    Startup can take several minutes. Logging is very verbose during startup. 
    Once logging activity is stopped, you can consider startup to be done.
    
    > To check startup of the 3 components is finished via the logs, checks the entries similar to these ones:
    >
    > For MongoDB (at the very beginning of the logs):  
    > ```
    > > mongo | 01:00:00 I NETWORK  [initandlisten] waiting for connections on port 27017
    > ```
    >
    > For the backend (at the very end of the startup logs):  
    > ```
    > > nlweb-backend | 01:00:00 [main] INFO  org.eclipse.jetty.server.Server - Started @59657ms
    > ```
    >
    > For the frontend (at the beginning of the startup logs):  
    > ```
    > > nlweb-frontend  | 01:00:00 [main] INFO  org.eclipse.jetty.server.Server - Started @17907ms
    > ```
    >
    
    2.3. Check the deployment via the web interface
    
    - Get the web interface port 
    
        If you did not configure it explicitly, default port is **80**.
    
        Read *1.2. edit the file according to your environment > Exposed frontend ports* previous section for more information.
        
    - Access the web interface
    
        Open your favorite web browser and access the web interface. Pattern is **http://myMachine:80**
    
        Check the login page is displayed. You must now create an admin user to be able to login.
        
    - Create an admin user
        
        // TODO
    
    2.4. Run a bench on your NeoLoad Controller (**optional**)
    
    - Configure your NeoLoad controller to access your NeoLoad Web.
     
       - Get *NeoLoad Controller REST port*.
      
            The default NeoLoad Controller REST port is **8080** unless you change it.
      
            Read *1.2. edit the file according to your environment > Exposed backend ports* previous section for more information.

       - Configure your NeoLoad Controller
       
            See [official documentation](http://www.neotys.com/documents/doc/neoload/latest/#24383.htm) for more information.

       - Run a bench on your NeoLoad Controller (transferring data to NeoLoad Web) and see the running bench on NeoLoad Web.

            See [official documentation](https://www.neotys.com/documents/doc/nlweb/latest/en/html/#25070.htm) for more information.

3. #### Stop  

    - 2 ways to stop NeoLoad Web properly:

        - On the terminal where you launch your NeoLoad Web, type CTRL C.
        
            **or**
        
        - On a terminal located on the working directory of your NeoLoad Web deployment:
        
            ```
            > docker-compose --file docker-compose-all-in-one.yaml down
            ```
    - Wait for the stopping to be finished:
    
        Process is finished when the output of your terminal displays: 
        
        ```
        Stopping dockernlweb_nlweb-frontend ... done
        Stopping dockernlweb_nlweb-backend ... done
        Stopping dockernlweb_mongo ... done
        Removing dockernlweb_nlweb-frontend ... done
        Removing dockernlweb_nlweb-backend ... done
        Removing dockernlweb_mongo ... done
        ```
    
## External MongoDB deployment

1. #### Configure

    1.1. get [docker-compose-external-mongodb.yaml](http://TODO)
    
    1.2. edit the file according to your environment

    - Setup access to your MongoDB
    
        Change ```environment``` section of ```nlweb-backend```.    
        **MONGO_HOST_VALUE** and **MONGO_PORT_VALUE** must be changed.
    
        ```
        environment:
            MONGODB_HOST: MONGO_HOST_VALUE
            MONGODB_PORT: MONGO_PORT_VALUE
        ```
    
        > 
        > Note for MongoDB requiring authentication.
        >
        > You can specify the login and password by adding the properties below:
        >
        >```
        >environment:
        >   ...    
        >   MONGODB_LOGIN: YOUR_MONGODB_LOGIN
        >   MONGODB_PASSWORD: YOUR_MONGODB_PASSWORD
        >```
        >
        
        > 
        > Note for MongoDB as a cluster of machines.
        >
        > You can specify the MongoDB URL of your cluster changing the **MONGODB_HOST** property value. You must also set **MONGODB_PORT** to **0** value.
        >
        >```
        >environment:
        >   ...    
        >   MONGODB_HOST: URL_OF_YOUR_MONGODB_CLUSTER
        >   MONGODB_PORT: 0
        >```
        >
    
    - Exposed backend ports (**optional**)
    
        *Same as the "All in one deployment" configuration*
    
    - Exposed front ports (**optional**)
        
        *Same as the "All in one deployment" configuration*

2. #### Run

    2.1. run the docker-compose file
        
    ```
        > docker-compose --file docker-compose-external-mongodb.yaml up --remove-orphans
    ```
    
    2.2. Look at the logs to check if the deployment is going fine (**optional**)

    *Same as the "All in one deployment" section (excluding MongoDB part)*
    
    2.3. Check the deployment via the web interface

    *Same as the "All in one deployment" section*
    
    2.4. Run a bench on your NeoLoad Controller (**optional**)

    *Same as the "All in one deployment" section*
    
3. #### Stop

    - 2 ways to stop NeoLoad Web properly:

        - On the terminal where you launch your NeoLoad Web, type CTRL C.
        
            **or**
        
        - On a terminal located on the working directory of your NeoLoad Web deployment:
        
            ```
            > docker-compose --file docker-compose-external-mongodb.yaml down
            ```
            
    - Wait for the stopping to be finished:
    
        Process is finished when the output of your terminal displays: 
        
        ```
        Stopping dockernlweb_nlweb-frontend ... done
        Stopping dockernlweb_nlweb-backend ... done
        Stopping dockernlweb_mongo ... done
        Removing dockernlweb_nlweb-frontend ... done
        Removing dockernlweb_nlweb-backend ... done
        Removing dockernlweb_mongo ... done
        ```

## All in one SSL deployment

1. #### Configure

    Deployment with SSL requires to have your own certificates.

    1.1. get [docker-compose-all-in-one-ssl.yaml](http://TODO)
    
    1.2. get [ssl.conf](http://TODO)
    
    1.3. create working directory
    
    - Put these 2 files in a working directory of your choice. 
    
      >Note: these 2 files must be in the same folder at the same level.
    
    1.4. certifications files 
    
    Put your certificate and key files in a specific folder.
    These two files must be explicitly named **nlweb.crt** and **nlweb.key**.  
    
    > For testing purpose only, you can generate self signed certificates with *openssl*.
    >
    > Command line example:
    > 
    > openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /PATH_TO_MY_FOLDER/nlweb.key -out /PATH_TO_MY_FOLDER/nlweb.crt
      
    1.5. edit the files according to your environment
    
    In *docker-compose-all-in-one-ssl.yaml*:
    
    - MongoDB data directory (**required**)
    
        *Same as the "All in one deployment" configuration*
    
    - SSL certification folder (**required**)
    
        Change *path/to/your/certs/folder* to the path of your certificates directory (where you put **nlweb.crt** and **nlweb.key**).
    
        ```
        nginx:
            ...
            volumes:
            - /path/to/your/certs/folder:/cert
            ...
        
        ```
    
    - Exposed backend ports (**optional**)
    
      3 ports are exposed with default values, change it if needed:
          
      - NeoLoad Controller REST port: ```8080```
      
      This is the port which will be used for communication between NeoLoad Controller and NeoLoad Web.
      
      Change the first value of the entry **8080:8080** below:
      
      ```
      nginx:
            ...
            ports:
                - 8080:8080
            ...
      ```

      - Public REST API port: ```8081```
      
      This is the [public REST API](https://www.neotys.com/documents/doc/nlweb/latest/en/html/#25050.htm) port which is exposed by default on the backend.
      
      Change the first value of the entry **8081:8081** below:
      
      ```
        nginx:
            ...
            ports:
                ...
                - 8081:8081
            ...
       ```
      
      - NeoLoad Web backend monitoring : ```9092```
      
      For monitoring purpose only, enable retrieval of information relative to the state of the backend services.
      
      Change the first value of the entry **9092:9092** below:
      
      ```
        nginx:
          ...
          ports:
              ...
              - 9092:9092
          ...
       ```
    
    - Exposed frontend ports (**optional**)
    
        2 ports are exposed with default values, change it to your needs if required:
              
        - NeoLoad Web interface: ```443```
          
        This is the port to be used to access the web interface.
        
        Change the first value of the entry **443:443** below:  
          
        ```
        nginx:
            ...
            ports:
                - 443:443
            ...
        ```
        
        - NeoLoad Web frontend monitoring: ```444```
          
        For monitoring purpose only, enable to retrieve information relative to the state of the frontend services.
        
        Change the first value of the entry **444:444** below:  
          
        ```
        nginx:
            ...
            ports:
                - 444:444
            ...
        ```

2. #### Run

    2.1. run the docker-compose file
        
    ```
        > docker-compose --file docker-compose-all-in-one-ssl.yaml up --remove-orphans
    ```
    
    2.2. Look at the logs to check if the deployment is going fine (**optional**)
    
    *Same as the "All in one deployment" section*
    
    2.3. Check the deployment via the web interface
        
    *Same as the "All in one deployment" section*
    
    2.4. Run a bench on your NeoLoad Controller (**optional**)
    
    *Same as the "All in one deployment" section*

3. #### Stop

    - 2 ways to stop NeoLoad Web properly:

        - On the terminal where you launch your NeoLoad Web, type CTRL C.
        
            **or**
        
        - On a terminal located on the working directory of your NeoLoad Web deployment:
        
            ```
            > docker-compose --file docker-compose-all-in-one-ssl.yaml down
            ```
            
    - Wait for the stopping to be finished:
    
        Process is finished when the output of your terminal displays: 
        
        ```
        Stopping dockernlweb_nlweb-frontend ... done
        Stopping dockernlweb_nlweb-backend ... done
        Stopping dockernlweb_mongo ... done
        Removing dockernlweb_nlweb-frontend ... done
        Removing dockernlweb_nlweb-backend ... done
        Removing dockernlweb_mongo ... done
        ```

## External MongoDB SSL deployment

1. #### Configure

    Deployment with SSL requires to have your own certificates.

    1.1. get [docker-compose-external-mongodb-ssl.yaml](http://TODO)
    
    1.2. get [ssl.conf](http://TODO)
    
    1.3. create working directory
    
    - Put these 2 files in a working directory of your choice. 
    
      > Note: These 2 files must be in the same folder at the same level.
    
    1.4. certifications files 
    
    Put your certificate and key files in a specific folder.
    These two files must be explicitly named **nlweb.crt** and **nlweb.key**.  
    
    > For testing purpose only, you can generate self signed certificates with *openssl*.
    >
    > Command line example:
    > 
    > openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /PATH_TO_MY_FOLDER/nlweb.key -out /PATH_TO_MY_FOLDER/nlweb.crt
      
    1.5. edit the files according to your environment
    
    In *docker-compose-all-in-one-ssl.yaml*:
    
    - Setup access to your MongoDB (**required**)
        
        *Same as the "External MongoDB deployment" configuration*
    
    - SSL certification folder (**required**)
    
        Change *path/to/your/certs/folder* to the path of your certificates directory (where you put **nlweb.crt** and **nlweb.key**).
    
        ```
        nginx:
            ...
            volumes:
            - /path/to/your/certs/folder:/cert
            ...
        
        ```
    
    - Exposed backend ports (**optional**)
    
        *Same as the "All in one SSL deployment" configuration*
    
    - Exposed frontend ports (**optional**)
    
        *Same as the "All in one SSL deployment" configuration*   

2. #### Run

    2.1. run the docker-compose file
        
    ```
        > docker-compose --file docker-compose-external-mongodb-ssl.yaml up --remove-orphans
    ```
    
    2.2. Look at the logs to check if the deployment is going fine (**optional**)
    
    *Same as the "All in one deployment" section*
    
    2.3. Check the deployment via the web interface

    *Same as the "All in one deployment" section*
    
    2.4. Run a bench on your NeoLoad Controller (**optional**)
    
    *Same as the "All in one deployment" section*

3. #### Stop

    - 2 ways to stop NeoLoad Web properly:

        - On the terminal where you launch your NeoLoad Web, type CTRL C.
        
            **or**
        
        - On a terminal located on the working directory of your NeoLoad Web deployment:
        
            ```
            > docker-compose --file docker-compose-external-mongodb-ssl.yaml down
            ```
            
    - Wait for the stopping to be finished:
    
        Process is finished when the output of your terminal displays: 
        
        ```
        Stopping dockernlweb_nlweb-frontend ... done
        Stopping dockernlweb_nlweb-backend ... done
        Stopping dockernlweb_mongo ... done
        Removing dockernlweb_nlweb-frontend ... done
        Removing dockernlweb_nlweb-backend ... done
        Removing dockernlweb_mongo ... done
        ```

## Advanced operations

- #### Logging with your NeoLoad Web deployment

    ##### 1. description

    By default, the terminal outputs the logs of each components of your NeoLoad Web.

    Each log entries are prefixed by the component names: **nlweb-frontend**, **nlweb-backend** and **mongo** (no mongo component if you use your own MongoDB).

    ```
    nlweb-frontend  | 01:01:01.0 [neotys-wsp-vaadin-access-ui-executor-58] INFO  ...
    nlweb-backend   | 01:01:01.1 [nioEventLoopGroup-4-2] INFO  ...
    mongo           | 01:01:01.2 I NETWORK  [initandlisten] connection accepted ...
    ```

    ##### 2. logging operations

    You can use the command ```docker-compose logs``` to display/manipulate logs ([official documentation](https://docs.docker.com/compose/reference/logs/)).
    
    2.1 Display the last 20 lines of logs for the front-end component (in *following* mode):
    
    On a terminal located on the working directory of your NeoLoad Web deployment:

    ```
    > docker-compose logs -f --tail=20 nlweb-frontend
    ```

    2.2 Output last 50000 lines of logs to a specific file:
    
    ```
    > docker-compose logs --tail=50000 > my-log-file.log
    ```

- #### Upgrade your NeoLoad Web deployment

    ##### 1.1. pull last updates of the components
  
    With your docker-compose deployment file, type the command:
    
    ``` 
    docker-compose --file YOUR_COMPOSE_FILE pull
    ```

    Example with the *All in one deployment*:
    
    ```
    > docker-compose --file docker-compose-all-in-one.yaml pull
    ```

    ##### 1.2. stop NeoLoad Web

    See previous section named *Stop*.
    
    ##### 1.3. start NeoLoad Web

    See previous section named *Run*.
    
- #### Backup your MongoDB data

    TODO
    
    
