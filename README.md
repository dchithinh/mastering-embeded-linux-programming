# Embedded Linux Programming Study Notes with QEMU

This repository contains my personal hands-on notes for bringing up an ARM
embedded Linux system with QEMU. The steps cover building a cross-compiler,
U-Boot, the Linux kernel, a root filesystem, and related boot setup.

These notes were written while studying *Mastering Embedded Linux Programming*.
For the full explanations and original material, please refer to the book.

# Build with docker
If your PC is running unbuntu with different version of 20.04 as this guide. To avoid errors(mismatch linux version, can not install requirement packages, etc) when building, there is a simple docker in this guide that can be used.
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

## Attribution

These are personal study notes and practical walkthroughs based on my own
hands-on work while studying *Mastering Embedded Linux Programming*.

This repository is not affiliated with or endorsed by the book author or
publisher. The original book content remains the property of its respective
author and publisher.

## License

Original notes, explanations, and repository-specific material in this
repository are licensed under the Creative Commons Attribution-ShareAlike 4.0
International License.

Some learning flow and technical steps were produced while studying *Mastering
Embedded Linux Programming*. The original book content remains the property of
its respective author and publisher.

See [LICENSE](LICENSE) for details.
