# Read me

The original idea is from  
https://medium.com/@gladevise/accessing-the-hosts-localhost-from-inside-a-docker-container-c5935e275953

We can achieve the same result with docker command too

```
docker container run --rm --name ubuntu -it --add-host host.docker.internal:host-gateway ubuntu bash
```