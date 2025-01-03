---
author: alex
category:
  - devnet
  - study-material
date: "2024-04-06T08:47:03+00:00"
guid: https://netwithalex.blog/?p=673
title: Dockerfile
url: /dockerfile/

---
### Introduction

Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. This page describes the commands you can use in a Dockerfile.

Docker runs instructions in a Dockerfile in order. A Dockerfile **must begin with a FROM instruction**.

### Example

```
# Use the official Python image as the base image
FROM python:3.9-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the Python script from the host machine into the container
COPY my_script.py .

# Run the Python script when the container starts
CMD ["python", "my_script.py"]
```

- ` FROM python:3.9-slim ` : Specifies that our image will be based on the official Python image with version 3.9, using the slim variant.
- ` WORKDIR /app ` : Sets the working directory inside the container to `/app`. Later instructions will be executed in this directory.
- ` COPY my_script.py . ` : Copies a file named `my_script.py` from the host machine into the `/app` directory of the container. The `.` refers to the current directory in the Docker context, which will typically be the directory containing the Dockerfile.
- ` CMD ["python", "my_script.py"] ` : Specifies the default command to run when a container based on this image starts. In this case, it runs the Python interpreter ( `python`) with the `my_script.py` script as an argument.

### **Table of Instructions**

FROM
ENV
COPY
EXPOSE
ADD
WORKDIR
RUN
LABEL
CMD

### Instructions in detail

**FROM**

Specifies the base image to build the container. Every Dockerfile must start with a FROM instruction.

```
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```

**ADD**

Specifies the origin of a local file or URL and transfers the associated files to the designated destination directory. If the source is compressed, like a .tar file, the ADD instruction will automatically extract the contents after the copy operation is finished.

```
ADD [OPTIONS] <src> ... <dest>
```

**RUN**

Executes shell commands within the image during the build process. This is typically used for tasks like package installation, environment setup, etc. Common commands include 'apt install' and 'pip install' for Python packages.

```
RUN [OPTIONS] <command> ...
```

**WORKDIR**

Sets the working directory for any subsequent RUN, CMD, ENTRYPOINT, COPY, and ADD instructions. It's better to use absolute paths instead of relative ones (../)

```
WORKDIR /path/to/workdir
```

**EXPOSE**

Functions as a type of documentation instruction that you can use to indicate the ports that will be used for the container. Default protocol is TCP, but you can define UDP as well.

```
EXPOSE <port> [<port>/<protocol>...]
```

**CMD**

Specifies the command to run when a container is started from the image. Can be overridden at runtime. You can only use it once.

```
CMD command param1 param2
```

**ENTRYPOINT**

Sets a main command that will always be executed when the container starts up.

```
ENTRYPOINT command param1 param2
```

**ENV**

Sets environment variables in the image.

```
ENV <key>=<value> ...
```

**VOLUME**

Creates a mount point and marks it as externally mounted. Used for persisting data outside the container.

```
VOLUME ["/data"]
```

**USER**

Sets the username or UID to use when running the container.

```
USER <user>[:<group>]
```

### Additional references

[Official Dockerfile documentation](https://docs.docker.com/reference/dockerfile/#user) \- Check this page for further details of the commands.

### [Go back to Blueprint](/study-devops-blueprint)
