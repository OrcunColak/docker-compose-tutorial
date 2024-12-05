# Read me

The original idea is from  
https://itnext.io/kubernetes-containers-and-the-lost-sigterm-signals-40007f35759a

In summary, if "shell" form is used to launch the process, SIGKILL is not directly sent to the launched child process

# SIGKILL

1. first, SIGTEM is sent to the PID 1
2. after 10 seconds SIGKILL is sent to the PID 1
3. if this did not help, then SIGKILL is sent to all processes in the container

The PID 1 is a special process, checking PID 1 shows us that it “recognizes” the only SIGHUP and SIGCHLD signals
and both SIGTERM and SIGKILL will be ignored by it

# The “Right way” to launch processes in a container

There are two ways for launching a container

1. INSTRUCTION [“executable”,”param1",”param2"] ("exec" form)
2. INSTRUCTION command param1 param2 ("shell" form)

This uses "shell" form. We have the /bin/sh as the PID 1, which calls Gunicorn through -c.

```
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENTRYPOINT gunicorn -w 1 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:80 app:app
```

Exec form. We will have only Gunicorn processes, which already can handle SIGTERM signals .
And now, if we send the SIGTERM to the PID 1, the container will finish its work normally:

```
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENTRYPOINT ["gunicorn", "-w", "1", "-k", "uvicorn.workers.UvicornWorker", "--bind", "0.0.0.0:80", "app:app"]
```