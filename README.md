# Containerize *.Net6* Web API using **Docker** 

### **What does containerization mean?**

Containerization is a form of virtualization where applications run in isolated user spaces, called containers, while using the same shared operating system (OS).

### **What is Docker Container?**

Docker is an open source platform that **enables developers to build, deploy, run, update and manage containers**—standardized, executable components that combine application source code with the operating system (OS) libraries and dependencies required to run that code in any environment.

------------
## About this exercise

### Previously:

- We developed a base structure of an api solution in Asp.net core that have just two api functions *GetLast12MonthBalances* & *GetLast12MonthBalances/{userId}* which returns data of the last 12 months total balances.
![](/After/assets/images/12m.jpg)


### In this exercise we will:

- Install Windows Subsystem for Linux (WSL)
- Install Docker for windows
- Configure *dockerfile* for our API
- Build a **docker image** of our API
- Run container using our image
- Create, run, push and pull our docker image in **Docker Hub**

--------------

### **Step 1: Installing WSL:**

The Windows Subsystem for Linux (WSL) is a feature of the Windows operating system that enables you to run a Linux file system, along with Linux command-line tools and GUI apps, directly on Windows, alongside your traditional Windows desktop and apps. See the about page for more details.

- Install it by entering this command in an **administrator** PowerShell or Windows Command Prompt and then restarting your machine
    ```powershell
    wsl --install
    ```
    ***Note:** The above command only works if WSL is not installed at all, if you run `wsl --install` and see the WSL help text, please try running `wsl --list --online` to see a list of available distros and run `wsl --install -d <DistroName>` to install a distro.

----------

### **Step 2: Install Docker Desktop:**

Follow these steps to install docker:

- Download [Docker Desktop for Windows](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)

- Follow the usual installation instructions to install Docker Desktop. If you are running a supported system, Docker Desktop prompts you to enable WSL 2 during installation. Read the information displayed on the screen and enable WSL 2 to continue.
- Start Docker Desktop from the Windows Start menu.
- From the Docker menu, select **Settings > General**.
![](/After/assets/images/2.jpg)

- Select the Use **WSL 2 based engine** check box.

- If you have installed Docker Desktop on a system that supports WSL 2, this option will be enabled by default.

- Click **Apply & Restart**.

That’s it! Now *docker commands* will work from Windows using the new WSL 2 engine.

------------  

### **Step 3: Configure *Dockerfile***


**What is `dockerfile` ?**

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image.

- Before starting, please note that you can use our existing API from *before* folder or create a new api using this command in the terminal 

    ```powershell
        dotnet new api
    ```

- Run your api using this command to see if your api is working. Make sure you run this command where `.csproj` is located
    ```
        dotnet run
    ```
    go to this url to check if our api is running. http://localhost:5070/api/Transaction/GetLast12MonthBalances/

- We will **publish** our API and use its artifact location later on. To publish your API run this command
    ```powershell
        dotnet publish -c release
    ```
    to see the location of this published artifact you can got to **bin > Release\net6.0 > publish**. These are all the files needed to host API and are necessary to export into container.

- Now we will create a new file named `dockerfile` in the same location where we have `BBBankAPI.csproj`. 
- Paste the below given code in this file 
    ```docker
        FROM mcr.microsoft.com/dotnet/aspnet:6.0 
        COPY bin/Release/net6.0/publish/ App/
        WORKDIR /App
        ENTRYPOINT ["dotnet", "BBBankAPI.dll"]
    ```
    - Here *mcr.microsoft.com* is our base image with runtime *.Net6*
    - It will **copy** the artifact from */bin/release/net6.0/publish/*  this folder and paste it to *App/* folder inside the container
    - Now */App* folder will be our **working directory**
    - `donet` command will be **Entry Point** into the container and will execute `BBBankAPI.dll` parameter

------------------

### **Step 4: Create our API's *Docker Image***

#### **What is Docker Image?**
A Docker image is a read-only template that contains a set of instructions for creating a container that can run on the Docker platform. It provides a convenient way to package up applications and preconfigured server environments, which you can use for your own private use or share publicly with other Docker users.

- To create docker image use this command 
    ```powershell
    docker build -t BBBank_API_image -f Dockerfile .
    ```
    *Note: there is a period ' . ' with a space after Dockerfile, it is not by accident and it is supposed to be like this

- To check the current working images run this command

    ```powershell
        docker images
    ```

    ![](/After/assets/images/3.jpg)

    You can see our currently created image named *bbbank_api_image* up and running. You can also see the running images from **Docker Desktop** we downloaded 
    ![](/After/assets/images/4.jpg)

    --------------

### **Step 5: Running a container from existing image**

#### **What are containers?**

Containers are packages of software that contain all of the necessary elements to run in any environment. In this way, containers virtualize the operating system and run anywhere, from a private data center to the public cloud or even on a developer’s personal laptop.

- To run a container use this command 
    ```powershell
    docker run -it -p 8000:80 --name {container-name} {image-name}
    ```
    - The `docker run` command **creates** a container from a given image and **starts** the container using a given command
    - `-p` it will map TCP port 80 in the container to port 8080 on the Docker host. 

- Now our api running inside container you can check it by running this command
    ```powershell
    docker ps
    ```
    ![](/After/assets/images/5.jpg)
 
 Here you can see that our container is running while using the image and port we have created. 
You can also see running containers from the docker desktop as well.

- Finally you can use this URL to see you API is running on port 8080. 
http://localhost:8000/api/Transaction/GetLast12MonthBalances/
![](/After/assets/images/6.jpg)

-------------

### **Step 6: Push/Pull Docker Images into DockerHub**

- Create an account at Docker Hub (https://hub.docker.com/) to be able to push and pull Docker Hub images.
- Use this command to login to you docker hub account you created 
    ```powershell
    docker login -u your_user_name 
    ```
- Password - The prompt will request our password for DockerHub
![](/After/assets/images/7.jpg)

- Before pushing our image to docker hub repository we first need to tag it by using this command
    ```powershell
        docker tag bbbank_api_image rajaastahir/bbbank_api_image
    ```
    We need to tag the image to send it to our own repository that is created when we sign up on hub.docker.com. It creates a default repository with our Docker ID.
- Finally to push the image into remote repository of docker hub use this command
    ```powershell
    docker push {user-name}/repository-name
    ```
    ![](/After/assets/images/8.jpg)

- You can also see your image in Docker Hub repositories in the account we created before.
![](/After/assets/images/9.jpg)
- Similarly to **pull** from your Docker Hub repository you can use this command
```powershell
    docker pull {user-name}/{Image-name}
```
- You can also directly `push` and `pull` from docker desktop app
![](/After/assets/images/10.jpg)

------
