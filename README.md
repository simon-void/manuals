# Manuals

Usefull tips/commands for
- Docker
- Docker Compose
- Hugo

## Docker

from https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide

**docker run $imageId ($command)** // create and start a container (starting-> invoking/(replacing) the start command)  
**docker ps (-a)** // show all running containers, -a -> all containers that ever ran  
**docker system prune** // remove all unused containers/networks/dangling images/build cache  

**docker create $imageId** // prepare filesystem/container but don’t start it yet, outputs a containerId  
**docker start (-a) $containerId** // (re)start a container, -a -> link output to current command  
**docker logs** $containerId // a record of all the logs from a terminated/running container is printed out/ the container isn’t restarted, it’s post-mortem  

**docker stop $containerId** // gives the container a bit of time to shut down/clean up, defaults to a kill after 10s  
**docker kill $containerId** // immediately terminates the container  
**docker exec -it $containerId $command**    // execute an additional command in a container, -it -> interactive terminal (else docker exec returns immediately) E.g.  
**docker exec -it $containerId sh**          // to start a shell and get full terminal access (alternativ: bash/powershell/zsh)

**docker build -t $tag**                     // tagging an image with tag: $dockerId/$projectName:$version(e.g. latest), builds Dockerfile  
**docker build -f Dockerfile.dev .**         // if you want to build a different file than Dockerfile use -f  
**docker commit -c $command $containerId**   // create image from container with command: e.g. 'CMD ["tomcat"]', see Dockerfile CMD  

**docker run -p $fromPort:$toPort $imageId** // port-forwarding from outside port to inside port, e.g. -p 8080:8080  
**docker run -v ${pwd}:/app $imageId**                      // create a volume from current folder to in-image /app folder  
**docker run -v /app/node_modules -v ${pwd}:/app $imageId** // make sure to not overwrite /app/node_modules folder when creating the volume (notice how that part doesn't contain a ':')  


### Dockerfile

E.g.

    FROM node:alpine
    
    WORKDIR /app           // any following command is executed relative to this path
    
    // copy from_path_outside_container_relative_to_buildCmd to_path_inside_container
    // to e.g. include configuration files
    // first ./ is current folder, ./ is relative to work directory inside the container
    COPY ./package.json ./
    RUN npm install
    COPY . .                 // splitting the copy-command is a build-time optimization when building an image, because
    // the 'npm install' step doesn't have to be repeated, if the package.json didn't change
    // it's taken from build-cache instead
    
    // Start command
    CMD [„npm“, "start"]

**docker build .**    // the . Is the folder context to use


### Prod Dockerfile

    FROM node:16-alpine as builder
    
    WORKDIR /app
    
    COPY ./package.json ./
    RUN npm install
    COPY . .
    
    // Start command
    RUN npm run build
    
    // same file, second phase
    FROM nginx
    COPY --from=builder /app/build /usr/share/nginx/html     // use build website from builder phase
    
    // nginx is started automatically in nginx container

// execute Dockerfile  
**docker build .**

## Docker Compose

E.g. *docker-compose.yml*

    version: '3'
    services:
        redis-server:
            image: 'redis'
        node-app:
            restart: always // restart policy (in case container terminates)
            build: .        // look in current directory after Dockerfile

---
// or use context to name different filename  

            build:  
                context: .
                dockerfile: Dockerfile.dev
---

            ports:
                - "8081:8081"  // local port to container port
            volumes:
                - /app/node_modules
                - .:/app
            command: ["npm", "start"]    // overwrite start command

Clients within a container can reference the service ids!
in this case:

    const client = redis.createClient({
        host: 'redis-server',
        port: 6379             // default-port for redis
    })

**docker-compose up**         // equivalent do docker run $imageId  
**docker-compose up -d**      // run in background  
**docker-compose up --biuld** // rebuilds the images and runs them  
**docker-compose down**       // stop containers  
**docker-compose ps**         // only show containers belonging to the local docker-compose.yml file  

### Restart Policies

what to do when a container terminates/crashed/is crashing

set in *docker-compose.yml*

    services:
        ...
        node-app:
            restart: "no"(default)/always/on-failure(is exit code other than 0)/unless-stopped


## Hugo

**hugo** // generate html from hugo templates  
**hugo server** // start a local server to hot-reloads the current site  
