# mastering-embeded-linux-programming
This is a complete guide to bring a embedded board using qemu steps by steps.
Start from building a cross compile toolchain, u-boot,linux and root filesystem.

For more details, The guide steps can be looked at the book: mastering embedded linux programming.
#

# Build with docker
If your PC is running unbuntu with different version 20.04 as this guide. To avoid errors(mismatch linux version) when building, there is a simple docker in this guide that can be used.
- Build docker:
    ```
    cd DockerBuild
    docker build -t build_machine_crosstoolng .
    ```
- Run docker in attach mode:
    ```
    docker run -t -d --name crosstoolNg build_machine_crosstoolng
    ```
- Enter the container in interact mode
    ```
    docker exec -it crosstoolNg bash

    Output:
    dummier@63e01c29ddaa:~$
    dummier@63e01c29ddaa:~$
    ```

Then you will be in the terminal, from there you can start to build with the command line followed this guide.
