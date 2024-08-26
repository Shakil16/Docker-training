    # Use an official Python runtime as a parent image
    FROM python:3.11-slim

    # Set the working directory in the container
    WORKDIR /app

    # Create a non-root user and group
    RUN groupadd -r appuser && \
        useradd -r -g appuser -d /app -s /sbin/nologin -c "Docker image user" appuser

    # Install dependencies
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt

    # Copy the current directory contents into the container at /app
    COPY . .

    # Change ownership of the application directory
    RUN chown -R appuser:appuser /app

    # Switch to the non-root user
    USER appuser

    # Specify the port the application will run on
    EXPOSE 8080

    # Run the application
    CMD ["python", "app.py"]


Explanation:
Base Image: The Dockerfile uses the official python:3.11-slim image, which is lightweight and suitable for production environments.
Non-Root User: A non-root user appuser is created, and the working directory is assigned to this user.
Dependency Installation: The requirements.txt file is copied into the container, and dependencies are installed using pip.
Application Files: All application files are copied to the /app directory.
Ownership Change: Ownership of the /app directory is changed to appuser to ensure the application can access the files.
User Switch: The USER directive switches the running context to appuser.
Port Exposure: Port 8080 is exposed, which can be used by the application.
Run Command: The application is started using the CMD instruction, running app.py.
Build the docker image
    docker build -t test1-user -f dockerfile.user .

Run the docker continaer
    docker run --rm -d --name app3 -p 8080:8080 test1-user:latest
Login into the container and check if you are able to swich to root directory.
    docker exec -it app3 bash
    cd /root
I should get  Permission denied 



#CMD

Purpose: Specifies the default command to run in a container.
Behavior: Can be overridden by arguments provided in docker run.
Shell form: CMD command param1 param2
Exec form: CMD ["executable", "param1", "param2"]
    FROM ubuntu
    CMD ["echo", "Default message"]

    docker build -t cmd-image1 -f dockerfile.cmd .
    docker run -it cmd-image1:latest
 Overrite thr CMD
    docker run -it cmd-image1:latest bash   
    docker run -it cmd-image1:latest ls -ltr
    docker run -it cmd-image1:latest ping www.google.com

#ENTRYPOINT
    FROM ubuntu
    ENTRYPOINT ["echo", "Default message"]

    docker build -t cmd-image2 -f dockerfile.entry .
    docker run -it cmd-image2:latest
-deafult message
    docker run -it cmd-image1:latest ls -ltr
-default message ls -ltr
Here is message appended with arument. 


    FROM ubuntu
    ENTRYPOINT ["echo"]
    CMD ["Default message"]

        docker build -t cmd-image3 --no-cache -f dockerfile.both .
        docker run -it cmd-image3:latest
output: default message
        docker run -it cmd-image3:latest hellow world   
output:hello worls, so it overrritd the arguments        

Key Differences
Override Behavior:

CMD: Can be fully overridden by providing a different command when running the container.
ENTRYPOINT: The command provided when running the container is appended as arguments to the ENTRYPOINT command.
Primary Use:

CMD: Best used to provide default arguments or a default command that can be easily overridden.
ENTRYPOINT: Best used when you want the container to behave like an executable and ensure that a specific command is always run.