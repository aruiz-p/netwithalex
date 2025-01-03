---
author: alex
category:
  - devnet
  - study-material
date: "2024-04-06T10:45:40+00:00"
guid: https://netwithalex.blog/?p=679
title: Docker Golden Images
url: /docker-golden-images/

---
### Introduction

The concept of golden image refers to an image that has been extensively tested and verified its fully operational. Following the same logic, an image that has not been tested or verified should not be considered a golden image. Organizations need to define the process to follow to mark an image as **golden**.

The process should include testing and verification methodologies. Also, from a security standpoint patches should be present and if there are external dependencies, it's crucial to update them to their respective recommended versions.

### Image Tags

Tagging Docker images plays a crucial role in managing and identifying different versions of images, including golden images.

#### Benefits

- **Identifying Versions** **-** Tagging allows you to assign meaningful names to Docker images, making it easier to identify different versions or variations of an image.
- **Version Control** **-** By tagging golden images with version numbers or release names, you can maintain a history of changes and track which versions have been tested and approved. You can roll back if needed.
- **Lifecycle Management** **-** Tagging helps in managing the lifecycle of golden images by providing an indication of their status and purpose ("dev", "qa", "prod")
- **Promotion Pipeline** **-** Tags can be used to signify the progression of images through different stages of testing and approval.

### Accessing Golden Images

Images should be available and accessible through trusted registries. One example is the Docker Hub which is regularly monitored and updated based on vulnerabilities, fixes, etc. It is possible to have private registries and base images as well.

#### Common Locations

- **Docker Hub -** Cloud-based service provided by Docker that serves as a centralized repository for Docker images. You can have both public and private registries.
- **Docker Trusted Registry (DTR) -** Enterprise-grade image storage solution provided by Docker. Extends the capabilities of Docker Hub to provide enhanced security, compliance, and collaboration features for organizations with strict security and regulatory requirements.
- **Private Docker registries -** Private Docker registries are self-hosted repositories for storing Docker images within an organization's infrastructure. Private Docker registries are typically deployed on-premises or within private cloud environments.

#### Image lifecycle

Typically, the steps to upload new images are the following:

- Create our Dockerfile
- Build the image
- Tag the image
- Push the image

#### Push Process

These commands are used to upload Docker images to a registry:

**docker login <URL>** **:** This command authenticates the Docker client with the specified Docker registry URL. It prompts you to enter your username and password for authentication. Once authenticated, Docker stores an authentication token locally, allowing you to interact with the registry without needing to log in again until the token expires.

```
docker login registry.example.com
```

**docker build -t <image\_name> -f <Dockerfile> .** **:** This command builds a Docker image using the specified Dockerfile ( `-f`) and tags ( `-t`) it with the given image name. The `.` at the end of the command indicates that the build context is the current directory, where the Dockerfile is located. For example:

```
docker build -t datastore -f Dockerfile_datastore .
```

**docker push <registry\_url>/<image\_name>:<tag>** **:** This command uploads the specified Docker image to the registry with the given URL. It includes the repository path ( `<registry_url>/<image_name>:<tag>`) where the image will be stored in the registry. Before pushing the image, ensure that you have tagged it correctly with the desired repository path. For example:

```
docker push registry.example.com/datastore:latest
```

By following these commands, you can authenticate with the Docker registry, build your Docker image, and then push it to the registry for storage and distribution. This process allows to share the containerized applications with others or deploy them to production environments.

### Additional References

[Docker Hub](https://docs.docker.com/docker-hub/)

[Docker Trusted Registry (DTR)](https://dockerlabs.collabnix.com/beginners/dockertrustedregistry.html)

[Private Docker Registry](https://docs.docker.com/registry/)

[Docker image build and push](https://docs.docker.com/reference/cli/docker/image/build/)

### [Go back to blueprint](/study-devops-blueprint/)
