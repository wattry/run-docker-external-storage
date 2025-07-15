# Run Docker External Storage in macOS 10.13+
Docker can be used to test projects without having to install and orchestrate software locally. However, if you're running a setup with a smaller internal drive it can be useful to move these stores externally. Newer Macs with smaller internal storage ~~soldered to the board~~ provide a challenge to developers. This tutorial will explain how to setup an external disk to store docker images and containers.

This solution was suggested in this "[Docker Forum Post](https://forums.docker.com/t/change-docker-image-directory-for-mac/18891/8)"

## Limitations 
The limited to MacOS 10.14: Mojave+ and with Docket Desktop. This tutorial assumes that the file locations are the same as the default locations used by Docker on macOS. Please create an issue if this does not work with a version of macOS or Docker.

## Setup 
This section will detail the process pre and post the installation of Docker

1. Setup the external drive's symbolic links.

    This step reproduces a similar structure to the internal storage, making it easier to debug or use documentation. You can use whichever directory structure you chose, just ensure you're consistent throughout the tutorial.

    * Using your shell or GUI create navigate to the root your external drive.
    * Create the following folder structure: 
    
    ```shell
     mkdir -p "Users/$(whoami)/Library/Containers"
    ```

    The p option on the mkdir command will, recursively, create the entire path's directory structure.

1. Create environment variables
    This step is optional however it will make it easier to reference the external location.

    Create/Open the config file for your shell. Newer Macs will use zsh by default though you will find a .zshrc or .bashrc. You can run `echo $SHELL` to find your default shell. These are suggestions and you can name your variables whatever you like. In this case $E stands for external and $EHOME for external home. These allow you to cd into and have a consistent source of truth, though this step is not required.

    ``` shell
    ## Env variables
    export E="/Volumes/<external>"
    export EHOME="$E/${HOME}"
    ```

1. Create Symbolic links
    ``` shell
    # Create a symlink to External Drive to preserve internal drive.
    ln -s "${EHOME}/Library/Containers/com.docker.docker" "/Users/$(whoami)/Library/Containers"
    ```

 1. Then either close and reopen your shell or reload your config with the following command

    ``` shell
      source .bash_profile
      or
      source .bashrc
      or
      source .zshrc
    ```

    Test your links and env variables
    ```shell
    $ echo $E 
    /Volumes/<external>
    $ echo $EHOME
    /Volumes/<external>/Users/<user>
    $  ls -la /Users/$(whoami)/Library/Containers/com.docker.docker
    lrwxr-xr-x   1 <user>  staff     74 Jan  1 00:00 com.docker.docker -> /Volumes/<external>/Users/<user>/Library/Containers/com.docker.docker
    ```
Your environment should now be ready.

## Pre Docker Installation 
  If you're using homebrew you can run `brew install docker-desktop` or download it [Download and install Docker CE for macOS](https://docs.docker.com/docker-for-mac/install/).

## Post Docker Install

This setup can be performed using cp/rsync or mv to get the com.docker.docker directory and content to your external drive. rsync will transfer the port files which mv will not (mv/cp will throw the following error *"Cannot listen: ~/Library/Containers/com.docker.docker/Data/vms/0/<your file\>: failed bind: Permission denied"*). This will not prevent you from copying as you can ignore this; these files are created when the docker daemon starts up.

1. Find where docker stores images: 

    Run the following command to see if Docker is running:

    ```
    launchctl list | grep docker
    -       0       com.docker.helper
    1949    0       com.docker.docker.2716
    ```

    1. If docker is not started yet, start it using the application launcher.
    1. Once it is running in the task bar secondary click on the docker icon at the top right of the screen.
    1. Select preferences and select the disk tab.
    1. Open a new shell
    1. Type or autocomplete the path up to the com.docker.docker directory part.
    1. If defaulted it should be located here *"/Users/\<user\>/Library/Containers/com.docker.docker"*.
    1. Stop Docker by secondary clicking on the docker icon and selecting *"Quit Docker Desktop"*

    To check that docker is no longer running use the following command: 
    ```
    launchctl list | grep docker
    -       0       com.docker.helper
    ```

2. Use mv or rsync to transfer the files. These steps may take a while to complete. Alternatively you can prune all the resources you do not need.

### Using mv 
```
$  mv -v "${HOME}/Library/Containers/com.docker.docker" "${EHOME}/Library/Containers"
```

### Using rsync 
```
$ rsync -a -v "${HOME}/Library/Containers/com.docker.docker" "${EHOME}/Library/Containers"
```

Once this is complete make sure all the files have transferred run this command. If there is no difference there will be no output. If the only are only socket files, you need not worry.

```
diff --brief -r "${EHOME}/Library/Containers/com.docker.docker" "${HOME}/Library/Containers/com.docker.docker"
```

You should now be able to use Docker on an external device.
