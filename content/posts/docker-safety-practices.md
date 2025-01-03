---
author: alex
category:
  - devnet
  - study-material
date: "2024-04-21T12:49:34+00:00"
guid: https://netwithalex.blog/?p=686
title: Docker - Safety practices
url: /docker-safety-practices/

---
## Introduction

Docker helps with packaging, distributing, and running applications. However, we should be responsible so we can ensure the security of the Docker environment. There are some best practices and strategies we can follow to achieve this objective.

## Security best practices and strategies

### No sensitive information on files

Dockerfiles are used to define the steps needed to build Docker images. It's crucial to avoid including sensitive information such as credentials, API keys, or other secrets directly in Dockerfiles. Keep in mind these files are often version-controlled and may be accessible to unauthorized users, resulting in a significant security risk.

Consider using environment variables or external configuration files to inject sensitive information into your Docker containers at runtime.

### Implement Role-Based Access Control (RBAC)

Role-Based Access Control (RBAC) is a security principle that restricts system access based on the roles and privileges assigned to users or groups. In the context of Docker, RBAC can limit the capabilities of containers and prevent unauthorized access to sensitive resources.

Consider implementing RBAC mechanisms such as:

- Using Docker's native Role-Based Access Control (RBAC) features to define granular permissions for users and services.
- Restricting container capabilities using Docker Security Profiles (Seccomp, AppArmor, or SELinux) to minimize the potential impact of security breaches.
- Regularly reviewing and updating access controls to ensure they align with the principle of least privilege and reflect changes in your environment.

### Use environment variables and files

Environment variables are mechanism for passing configuration information to Docker containers. They allow us to decouple application configuration from the container image, so we can customize behavior across different environments without modifying the underlying image.

When using environment variables, we need to follow security best practices:

- Avoid hardcoding sensitive information directly into Dockerfiles or source code.
- Use encrypted secrets management tools to securely store and retrieve sensitive data.
- Limit access to environment variables containing sensitive information to authorized users and services.

We can use Docker's built-in support for environment files ( `--env-file`) to load environment variables from external files.

```
###
Content of my_env_vars.env
###

DB_HOST=localhost
DB_USER=admin
DB_PASSWORD=secretpassword
```

Dockerfile consuming the file

```
# Dockerfile

FROM ubuntu:latest
COPY . /app

# Load environment variables from file
ENV_FILE=my_env_vars.env
RUN set -o allexport; source $ENV_FILE; set +o allexport

CMD ["./app"]
```

To run build and run

```
# Build the Docker image
docker build -t myapp .

# Run the Docker container with environment variables from file
docker run --env-file my_env_vars.env myapp
```

## [Go back to Blueprint](/study-devops-blueprint)
