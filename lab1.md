# Lab 1
Tools and Software

This lab will walk you through the tools we will use in this class and the best practices for running the labs.

## Make Dockerfile
First we will make a dockerfile so that we can build an image for this project.

- Make a dockerfile called `dockerfile`
- Make the dockerfile use the `tensorflow/tensorflow:2.3.0-gpu` container as the base 
- In the dockerfile install the ubuntu package `graphviz` and `python3-tk`
    - Whenever you install packages you always need to update first in the same line. If you do not update then if you build the image later it will run into errors because the packages will be out of date. Here is the dockerfile example lines of code to install packages: 
        
      ```dockerfile
      RUN apt-get update &&  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
            graphviz \
            python3-tk \
        # now in the same command clean up the pacakges to shrink layer size
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*
      ```
    - Note sometimes packages will not install because they wanted user intput. In this case need to put `DEBIAN_FRONTEND=noninteractive` before the `apt-get install`. For example the package `python3-tk` (which allows figures in `matplotlib`) will ask for user input if you do not include this line. In case you are confused on bash syntax the `&&` is to run multiple commands with the same line. Each line needs a `\` at the end if you want the previous line to continue. The last two lines are to cleanup the `apt-get` commands earlier:
    
    ```dockerfile
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*
    ```
   
    
- Make a `requirements.txt` that contains the python packages for `sklearn` and `matplotlib` and `pandas` each on a different line. Copy that file into the image. Also install these requirements using the `python -m pip` command like so:
```dockerfile
RUN python3 -m pip install --upgrade pip \
    && python3 -m pip install --no-cache-dir -r requirements.txt
```
- Note `python -m pip` is intalling pip packages and making sure our `python` environment is where they get installed. Usually this is the same as just saying `pip`.

### Extras
All the lines above we will use for every lab in some way. The next lines are to illustrate the difference between `ARG` and `ENV`.

In the Dockerfile add the following:
- Add a line of code in the dockerfile to add an environment called `SOMENUMBER_ENV` and make the default value 3.
- Add a line of code in the dockerfile to add an argument called `SOMENUMBER_ARG` and make the default value 5.
- Make a shell script called `echoscript.sh`. The script will echo (print to the terminal) the above variables using the following contents
```bash
echo ENV: ${SOMENUMBER_ENV}
echo ARG: ${SOMENUMBER_ARG}
```
- Add a line of code in the dockerfile to copy the above shell script into the container 
- Add a line of code to run the shell script during the build phase using the `RUN` keyword
- Add a line of code to run the scell script as the default command using the `CMD` keyword

## Run Docker Image
Usually we will not run docker image directly but we will once to show how it is done.
- Make a shell script called `buildandrundocker.sh` that builds the docker image in one line. Usually you would just run the command on the command line but I need to see what you actually were running. 
- Give the image the tag `lab1:yourlastname`. 
- Make `buildandrundocker.sh` also run the image after the image is build. The container should automatically run the `echoscript.sh` when started due to the `CMD` line added in the `Dockerfile`.

Run `buildandrundocker.sh` and briefly explain the echo output in a comment in the `buildandrundocker.sh` file. You should compare and contrast the argument and the environment variables from the two times we ran  `echoscript.sh`. 

## Docker Compose
We will usually run all our docker containers using `docker-compose`. This will walk through the steps to make a docker compose file
- Make a docker compose yaml named `docker-compose.yml`
- Add the version number 3 to the top like so `version: '3'`
- Make a `services` group that has in it a service named `lab1`
- Make a `build` group in our service `lab1`. 
    - The build group should have `context .` which means build from the current folder.
    - The build group needs to tell which dockerfile we are using to build like so `dockerfile: dockerfile` 
    - In the build group add an 'args' group an change the value of `SOMENUMBER_ARG` to 6.
- In the service group make an `environment` group.
    - In the `environment` group make an environment variable `SOMENUMBER_ENV` with a value of 4. 
    - Add an environment variable `DISPLAY` from the host computer with the line `DISPLAY:`. 
    - Note that we gave a value to `SOMENUMBER_ENV` but we let the host system pass through the value of `DISPLAY`. To see what is being passed you could open a terminal and type `echo $DISPLAY` to see what your host system has for `DISPLAY`. The `DISPLAY` environment variable will let us display figures in `matplotlib` from our container
- Just as an example pass through port `8000` on the host machine to `8001` on the container. In future labs we will not need to do this.
- Pass through your user id and group id as the user/group inside the container for example `user: 1000:1000`. 
    - Your user id will usually be 1000 and so will be the group. To verify this run `id -u` and `id -g` to check. 
    - This will prevent any files we write later from being owned by the root user and thus requiring you to change the ower later. 
- Make a `volumes` group under the service so we can mount some folders. 
    - Mount the folder `/tmp/.X11-unix` on the host to the folder `/tmp/.X11-unix` on the container. This will help us plot figures in `matplotlib`
    - Mount the folder `/etc/passwd` on the host to the folder `/etc/passwd` on the container. This will help certain programs know your username from your user id you set earlier.
    - Mount the folder `/opt/data` on the host to  `/opt/data` inside the container. This spot on your host machine is where you will store future lab data so make sure the drive it is on has some space. You may not want to put all your data there at `/opt/data`. If so make a folder on your host machine wherever you want called `data`.  Now make a link at `/opt/data/` that goes to that folder. For example `ln -s /home/yourusername/path/to/your/folder/data /opt/data`.  This ensures all our data will be in the same spot for any host computer we run on.
    - Mount the current folder `.` on the host to `/opt/project` inside the container. This is the standard place for our project code to be.
- Set the default working directory to the `/opt/project` folder ide the container.
- Make the `command` argument of the docker compose run `echoscript.sh` from above using `/bin/bash`.

Run `docker-compose build` (build the container even if it already exists) and then `docker-compose up` to run the docker compose. Briefly explain in a comment the `buildandrundocker.sh` file the difference between when you ran the docker container without the compose changing values. 
 
## Python
Next we will write a python file called `lab1.py` that will run in the docker container we just made.
The python file will do the following:
1. Make a list from one to ten and initialize to all ones
1. Make the first entry in the list a zero
1. Calculate the first 10 fibonacci numbers (starting with 0, 1) using a loop and put them in the previous list
1. Print the fibonacci numbers to the console

## PyCharm Project
Put all of this in a python PyCharm project. Put the python code in a folder named `src`. From in PyCharm right click on the `src` folder and go to `Mark Directory as` then click on `Sources Root`.
All other files goes on the root level.

Turn off Scientific Plotting. Go to `File` -> `Settings`. On the left side expand `Tools` and click on `Python Scientific`. Uncheck the box `Show plots in tool window`. This would prevent all the setup we did from showing `matplotlib` figures. 

Setup Pycharm to use the `docker-compose.yml` file from eariler (See user instructions on the [nhl_instructions](https://git.antcenter.net/jcurro/nhl_instructions/blob/master/UserInstructions.md) repo if want a detail step by step.) 


# Make the lab easier to grade
In the interest of making the lab easier to grade we will always make the `docker-compose.yml` of the lab able to run and do all work required simply by running `docker-compose up` in the root folder. The command I will run to check the lab code will be:
    
```bash
docker-compose up --build labX
```
This command mounts the project and data folder automatically.

You will have to modify the `command` of the service in the `docker-compose.yml` to run the main python code for the project. For Example `command: python src/lab1.py`. For lab1 keep the old command as a comment but make this the final one that runs.

When you use any data in your lab (usually provided by the instructor) do not modify the data as given by the instructor. Have your code expect all the data to be in `/opt/data` inside the container. This way the code can run on any computer without modifying the data. You will need to make processed data in future labs so put all processed data in a folder with your name like `/opt/data/eeng645/lab1/yourname` or the project folder (but don't commit them to GitHub!) so your code does not fail because of data from other students. Thus, your code should run from a fresh empty data directory and make all necessary processed data and folders besides the given data for the lab. 
## Lab1 specifics
For lab1 I will specifically be running the following commands (ie. make sure these commands work)
```bash
# check the build and run script
/bin/bash buildandrundocker.sh
# check the docker-compose and python
docker-compose up --build
# check user added multiple commits
git log
```
I will also look at other files for comments as noted in the lab so these commands are necessary but not sufficient to complete the lab.

## Git
For good programming practices and to make it easier to grade students will also push their code to the GitHub repository. 

To save space on our GitLab server never put large files in the git repository. Large files are around 100s of MB or larger. For example do not upload training data or models. Uploading figures is alright but please try not to make very large figures.

In order to ignore large files and other nusiance files we will add a file called `.gitignore` to the project root. We will add `*.pyc` and `.idea` to the `.gitignore` file which will ignore all `.pyc` files and the `.idea` folder which is PyCharm's project for your personal project settings. Always make the `.gitignore` before adding and committing files.

### Add and Commit Files
Use the GitHub Classroom assignment for each lab to submit your final code for each lab. Clone the template example with your name made automatically and modify the code for the lab (commit often!). In order to show that you can make new commits add a comment line to the python script with the text `new commit text`. Then add then commit that change and push that as well. This shows that you know enough to clone, add, commit, commit again, and then push a repo. 
