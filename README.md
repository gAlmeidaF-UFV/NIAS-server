# NIAS-server

A schematic describing the server is illustrated below. 

![](images/Imagem-servidor.png)

## Overview

In this section we will follow the steps of general server operation, this server will support python and MATLAB programming languages:

1. The researcher must build a directory called ***project***, inside his personal computer, containing all the files necessary for his project to run:
   - this directory must have two other directories inside of it, they must be called ``code`` and ``input``
   - Under ``code`` will be the ``app`` file, which will be the executable program, and ``requirements.txt``, intended to indicate the libraries used in the code.
   - The directory ``input`` will contain the data by the program as input, such as images or datasets

2. In order to run his project, the researcher must use ssh to get inside the server in his respective user. For doing that, the following command must executed:
   - To configure ssh in the PC, the resercher can use these links, to configure in [windows](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse) and to configure in [linux](https://ubuntu.com/server/docs/service-openssh) 
   ```
   ssh -p PORT user@SERVERIP
   ```
   - Once the researcher enter in his user, he must execute the [`job`](job.sh) file (Described below) using the following command:
   ```
   ./job
   ```
3. With the comands in step 2, the server will copy the project file from the researcher's PC and create a new directory fot it within the user's home directory

4. After copying the project file, ter server will put the apropriate Dockerfile ([Python](dockerfiles/dockerfile-python) or [MATLAB](dockerfiles/dockerfile-matlab)) from the templates directory for the project directory

5. Using the Dockerfile and the project file, the server will build an image and run a container from it, generating output to the code
   - It is important that all the output which the researcher wish to have access must be send to a directory called `/output`, because this is the directory inside the container that is attached to the real storage within the server

6. Finally, the output generated by running the project container will be sent back to the researcher via ssh protocol
   - A directory called `output` will be created within the project directory on the researcher's PC to receive the project outputs

## User Registry

First of all, to use this server, the researcher must register it self in the server. To do so, one must ask for the server administrator to run the [user registry](user-record.sh) file. this file works as follows:

  1. Informations of the new user must be provided, such as:
     - A user name, made as follows: firstname-lastname
     - The name of the user in the researcher's personal computer
     - The port that reponds to ssh in the researcher's personal computer
     - A passphrase for building the ssh key
     - the UFV-VPN IP address of the resercher's PC (This IP address must be fixed)
     
  2. With the above information, a new user will be created, he will be part of the docker group in addition to his own. the user home directory will use [skel-client](skel-client) to be build, this directory will contain the [job](job.sh) file (described later) and .ssh directory.
 
  3. Inside [ssh](skel-client/.ssh) directory, the researcher's PC will be registered as a ssh host, in order to be accessed later, during the [job](job.sh) routine

  4. To finish the registration, the server adminitrator must get in the newly created user and execute the commands to generate a new ssh key, using the passphrase provided, and send the public key to the researcher's PC  

## Job
Once the researcher is registered, the server will be available for him to run his codes, for this, the script [job](job.sh) must be executed. this section will explain how it works

1. Some informations of the new job must be provided
   - Name of the user who is creating the job
   - Name of the project which will run in the server
   - The path of the project on the researcher's personal computer
   - In what program language the project is writen, Python or MATLAB

2. A new directory is created under the user home directory whose name is the name of the project
   - via ssh protocol, the project file is copied from the researcher's PC to this directory on the server 

3. A pertinent Dockerfile (Python or MATLAB) is copied from the templates directory to the project directory on the Server

4. Now the script uses the docker build and run commands to build a new image, named after the project, create a container and execute it
   - in this command, the `/output` directory inside the container is bound to an output directory in the project directory, since the latter is in actual server storage

5. After the container runs and its output is generated, the scrips sends, via ssh, this output back to the researcher's PC, creating an output directory in the previously given path

## Dockerfiles

This server runs its jobs using Docker containers, as described earlier, the images are built from the code that the researchers make. In the current section the python and MATLAB dockerfiles will be described.

1. There are two diferent dockerfiles, one for python projects and the other for MATLAB, both are similar with minor differences.

2. The MATLAB dockerfile uses the r2021a version as its basis, while the python dockerfile uses 3.8-slim-buster

3. In the MATLAB dockerfile the license path is given as environment vriable

4. three directories are created, these are `/code`, `/input`, `/output`

5. The content of the code and input files within the project directory are copied to `/code` and `/input` directories, respectively.

6. In python dockerfile, the necessary libraries are installed according to `requirements.txt` file

7. The researcher's code is executed

## Next Steps

This is a first version of the server, therefor, there is a lot of room to improve and adjust some possible issues. this section will talk about some of these possible improvements and adjustments. 

 - [ ] Analyze possible problem thet researchers are likely to have with opening ports on their routers
 
 - [ ] Use a second machine connected to the current one to build a cluster
 
 - [ ] Use pipelines to allow user to use send new jobs for the server only by commiting to github  
