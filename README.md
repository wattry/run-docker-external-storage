# Run Docker External Storage in macOS 10.13+
Docker can be useful to test projects without having to install software locally to test components. However, it is resource hungry. Newer machines with smaller internal storage provide a challenge to developers. This tutorial will explain how to setup an external disk to store docker images and containers.

This solution was suggested in this [Docker Forum Post](https://forums.docker.com/t/change-docker-image-directory-for-mac/18891/8)

## Limitations 
The limitations of this tutorial are that it is only applicable to macOS and specific the following to versions: 10.13: High Sierra (Lobo) and 10.14: Mojave. Part of this would assume that the file locations are the same as macOS docker defaults.

## Setup 
This section will detail the process pre and post the installation of Docker. There are a few steps you can go through before you move on.

1. Setup your external drive to create symbolic links.

    I prefer to try reproduce my internal storage's file structure to make it easier to access file locations.

    * Using your shell (I use bash) or GUI create navigate to the root your external drive.
    * Create the following folder structure: 
    
    ```
     mkdir -p Users/\<user\>/Library/Containers
    ```

    The p option on the mkdir command will create the entire path's directory structure.

2. Create environment variables and symbolic links

    Create/Open config file for your shell. Bash uses .bash_profile, so I used this. These are suggestions you can name your variables whatever you like. $E stands for external and $EHOME for external home.

    ``` bash
    ## Env variables
    export E=/Volumes/<external-name>
    export EHOME=$E/$HOME
    
    ## Symbolic links

    # Create a symlink to External Drive to preserve internal drive.
    ln -s $EHOME/⁨Library⁩/⁨Containers⁩/com.docker.docker /Users/<user>/Library/Containers
    
    ```

 3. Then either close and reopen your shell or reload your config with the following command:

    ``` bash
        source .bash_profile
    ```

    Now let's test your links and env variables:

    ``` 
    $ echo $E 
    /Volumes/<external-name>
    $ echo $EHOME
    /Volumes/<external-name>/Users/<user>
    $  ls -la /Users/<user>/Library/Containers/com.docker.docker
    lrwxr-xr-x   1 <user>  staff     74 Dec  1 14:47 com.docker.docker -> /Volumes/<external-name>/Users/<user>/Library/Containers/com.docker.docker
    ```
Your environment should now be ready.

## Pre Docker Installation 
1. [Download and install Docker CE for macOS](https://docs.docker.com/docker-for-mac/install/).

## Post Docker Install

This set involves either using cp/rsync or mv to get the com.docker.docker directory and content to your external device. I used both, but rsync will transfer the port files which mv will not (mv/cp will throw the following error *"Cannot listen: ~/Library/Containers/com.docker.docker/Data/vms/0/<your file\>: failed bind: Permission denied"*). This will not stop you from copying. You can ignore this as these files are created when the docker daemon starts up. My preference would be to use mv, since it is faster.

1. Find where docker is storing images: 

    Run the following command to see if Docker is running:

    ```
    launchctl list | grep docker
    -       0       com.docker.helper
    1949    0       com.docker.docker.2716
    ```

    1. If docker is not started yet, start it using the application. 
    2.  Once is is started secondary click on the docker icon in the top right of the screen.
    3.  Select preferences and select the disk tab. 
    4.  Type or complete the path up to the com.docker.docker directory part.
    5.  It should look like this if it has used defaults: 6. *"/Users/\<user\>/Library/Containers/com.docker.docker"*.
    6.  Stop Docker by secondary clicking on the same docker icon and selecting *"Quit Docker Desktop"*

    To check that docker is no longer running use the following command: 
    ```
    launchctl list | grep docker
    -       0       com.docker.helper
    ```

2. Use mv or rsync to transfer the files. These steps may take a while to complete, be patient. 

### Using mv 
```
$  mv -v $HOME/Library/Containers/com.docker.docker $EHOME/Library/Containers
```

### Using rsync 
```
$ rsync -a -v $HOME/Library/Containers/com.docker.docker $EHOME/Library/Containers
```

Once this is complete make sure all the files have transferred run this command. If there is no difference there will be no output. If the only are only socket files, you need not worry.

```
diff --brief -r $EHOME/Library/Containers/com.docker.docker $HOME/Library/Containers/com.docker.docker
$   
```
