# Builder Docker image

Docker image for building projects.

Move to the directory where the `Dockerfile` is located.
```sh
docker build -t pvsneslib .
```

You can explore the content of the generated image using.
```sh
docker run --rm -it --entrypoint /bin/bash pvsneslib
```


