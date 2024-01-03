# Docker from Scratch
> Written by Noah Smith for COMP 423: Foundations of Software Engineering.
> *Last Updated: 12/28/2023*

## Preface
"So what's this whole Docker thing anyways, and is it even really that important?"
Fair question, and while the answer to that first part is a little complicated, the answer to the second part is definitely **YES**, it is that important!

Docker is a key technology, both to this course and out in industry, and is used heavily by many companies. Within this course, it's what powers tools including Dev Containers and Kubernetes/OpenShift, so understanding Docker is key to understanding them, too. In industry, even if you aren't using these exact technologies, you'll often still find Docker being used in some way or another, and knowing how to use it will come in handy.

Anyways, now that you're (hopefully) convinced, let's dive in!

## Docker Basics
What is Docker? Basically put, **Docker is a tool for creating and running containers**. So then, what's a container?

Imagine the following scenario: You've just finished writing a program, and now you're ready to deploy it to a server. There's just one problem: That server's environment is totally different from yours! Let's say your program uses a specific version of Node and TypeScript, relies on some dependencies that need to be installed separately, and needs to be built in a certain way before you can run it. While your personal machine has all of these requirements satisfied, if you wanted to run this on the server, you'd need to manually go through all these steps and set it all up again.

Wouldn't it be easier if you could create a snapshot of an environment with everything you needed perfectly set, including all of your program files, and then run that entire thing in a self-***contained*** environment on the server, or any other computer for that matter?

That's where Docker comes in. In Docker jargon, the "snapshot" of the environment is called an **image** and the "self-contained environment" that runs that image is the **container**. Or, flipping those definitions, **a container is a running instance of an image** and an **image is a snapshot of an environment**. By these definitions, you can see that an image must exist first before a container ever can. So then, how can we create an image?

## Docker Images
Well, an image is just supposed to be a snapshot of an environment, right? So couldn't you create one by just freezing your computer in time and copying all of its contents over? It's a good idea, but unfortunately, that won't quite work - for one, that would take up way too much space. Instead, we'll want to distill only the ***key aspects*** of our environment that we need to run our program, and build an image from there.

Luckily, Docker has a simple tool for doing exactly this: The **Dockerfile**. A Dockerfile is just a step-by-step guide to creating an image, where you specify exactly what files to put where, what commands to run, and a few other things. It typically lives in the root directory of your project, and is simply called "Dockerfile" without the quotes.

Going back to our previous scenario, let's lay out the exact steps we'd need to execute in a brand new environment to get our program working.

 1. Install the correct version of Node. Let's say we're using version 20.
 2. Copy our program's files over.
 3. Install our dependencies.
 4. Build our program.
 5. Run it.

For the sake of this example, there exists a repository with a sample program already written linked [here](https://github.com/noahsmiths/Docker-Hello-World-Example). If you want to try Docker out and follow along, clone the repo and edit the included Dockerfile according to the following steps.

Now, let's translate those instructions into an equivalent Dockerfile!

## Dockerfile

### 1. Install the correct version of Node.
Now, you might be thinking: If we're building this image from the ground up, what's our starting point? What are we even building from? The answer is a **base image**! An image is just a snapshot of some environment, so what we can do is take one that already exists as a base, add on our changes, and then we'll have an image that does exactly what we need. Luckily for us, the good people over at Node made us an image conveniently named `node` that includes everything you need to run a Node program. Specifically, we want to use version 20, so we can append it to the name with a semicolon, like this `node:20`.

Now, in our Dockerfile, we use the `FROM` instruction to tell Docker what image to use as the base, like this:

```dockerfile
FROM node:20
```

There we go, the first line of our Dockerfile, done!

### 2. Copy our program's files over.
After specifying our base image, we must next specify where we want to make our changes. At this point, you can think of this as essentially being in a terminal that is, by default, at the root directory `/` of our base image. There are already a lot of other folders here, so lets create a new one just for our project called `/app`, and then switch into that. In a regular command line, that would look like two commands: `mkdir /app` to make the directory, and `cd /app` to navigate into the directory. In a Dockerfile, it's combined into one instruction called `WORKDIR`, so we would add the following line:

```dockerfile
WORKDIR /app
```

Next up, we need to copy the files over themselves. To do this, we use the `COPY` instruction, which takes two arguments: The source directory which is a path on our computer, and the target directory which is a path in our image. Our source directory is specified relative to where the Dockerfile is located, and since it's in the same directory as the rest of our program files, we can just use `.` for the path. Similarly, because we're already located in `/app` in our image, we can specify the current directory as the target with `.` Combined, this results in the following:

```dockerfile
COPY . .
```

Great! Now, we have all of our source files in our image.

### 3. Install our dependencies.
With projects in Node, all of our dependencies are stored in the `package.json` file, so all we need to do is run the command `npm install` in the same directory and our program's dependencies will be taken care of. The instruction we use in our Dockerfile is `RUN`, followed by the command. Go ahead and add this into your Dockerfile.

```dockerfile
RUN npm install
```

But wait, there's more! While it's not included in our `package.json`, we'll also need to install typescript so we can compile our program. We can do that with the command `npm install typescript -g` (the `-g` flag tells npm to install it globally so we can use it anywhere). Add this in to your Dockerfile, too.

```dockerfile
RUN npm install typescript -g
```

### 4. Building our Program.
To build a TypeScript program, we'll need to invoke the typescript compiler we just installed through the `tsc` command. Similarly to before, we just use the `RUN` instruction again.

```dockerfile
RUN tsc
```

This will produce a file called `index.js` in our image, which is a compiled version of the `index.ts` file and ready for execution.
After this command, our files will be set up exactly as we need them, and we won't need any more configuration on that front.
### 5. Run it.
Wait, I thought containers ran, not images! Why is this included in our Dockerfile?

You're correct! Images themselves don't run, they're run by containers. However, when we tell create a container to run an image, it needs to know *what to run*. That's what we're going to specify here. The Dockerfile instruction for this is called `CMD` for "command", and it'll be the first thing the container runs when we create it. As such, **there can only be one `CMD` instruction in a Dockerfile**.

Syntatically, it's just like the `RUN` instruction, so just as we'd run our program with `node index.js` on our computer, we'll specify the same in the Dockerfile.

```dockerfile
CMD node index.js
```

Now, we're basically done. We can stop right here and our Dockerfile will function just fine. However, there's a final instruction that, while optional, is a good one to include when needed.

Specifically for this example, our server is going to listen on port `3000`. However, Docker has no idea about this unless we tell it explicitly that our application will use that port, which we can do with the `EXPOSE` instruction. This instruction's sole purpose is to tell Docker, "Hey, the program in this container is listening on port 3000," which can then be seen by anybody else who might use your image down the road. You can think of this instruction as more providing some handy documentation than actually completing a task.

```dockerfile
EXPOSE 3000
```

And there you have it, your Dockerfile is done and you're ready to build your image!

## Building an Image
Just a quick recap: We just finished the Dockerfile stage, meaning we can now build an image from it. After that, we'll have a usable image from which we'll be able to start a container, and see our app in action!

Luckily, at this point all that's left is a few commands. The first one we'll need is `docker build`, which will use our Dockerfile to create an useable Docker image. To do this, go to your terminal and make sure you're in the same directory as your Dockerfile.

The `docker build` command only requires one argument: The directory containing our Dockerfile. Since we just navigated our terminal to the same directory as the Dockerfile, we can use `.` to denote our current directory.

However, there are some other optional arguments we can pass, which will make our lives a lot easier down the road. For the sake of this example, we'll use the `--tag` argument, which allows us to specify a tag, or a name, for our image. The tag I'll use here is `my_server`.

All of these together result in this command:

```bash
docker build --tag my_server .
```

Go ahead and run this, and you should be able to see some pretty interesting things. The first time you run it, this will probably take a minute or two as Docker will need to download the `node:20` base image. After that, though, you'll see Docker then execute your Dockerfile line by line all but stop before the `CMD` instruction. Remember, this happens because the `CMD` instruction is telling the container what to run, not the image.

You'll also notice a lot of strings that start with `sha256:` and have a long string of letters and numbers after them. These are called hashes, and are actually how Docker keeps track of images under the hood. Hopefully, this sounds familiar to you because this is very similar to how Git works! **Just as Git uses a hash to track every commit, every Docker image also has a unique hash. And, just as Git keeps a sort of "linked list" from commit to commit, so too does Docker with its images!**

That's why we used the `--tag` option above. What a tag really allows us to do is to have a human-readable label reference a hash. Without it, we'd need to identify our image directly by its hash which is a very ugly mess of random characters. We've actually already seen and used a tag before, too! `node:20`, which we used for our base image, is actually just a tag itself that points to a hash of a publicly available image.

Going back for a moment, now it should be clearer how our Dockerfile and build process actually work. Docker found the hash of our base image referenced by tag `node:20`, pulled that image, and then executed our Dockerfile line by line. **Each step of the Dockerfile actually created a new intermediate image with its own hash, called a layer**, which is what you saw during the build process. **After we're done with all of our layers, we create an output image from all those layers combined, which gets its own hash and is what the `my_server` tag references.**

If you'd actually like to see all the Docker images on your system and their corresponding hashes, you can run `docker images` and look at the `IMAGE ID` column.

If all of this seems like a lot at once, it's because it is. If you need to re-read it, make sure to focus on the bolded areas and get those down, and remember these key takeaways:

1. Docker keeps track of images by using a hash, just like Git. Tags are just human readable names that reference a hash.
2. When images are built, it's done by executing each instruction of the Dockerfile in order.
3. Each one of these instructions creates a new intermediate image called a layer, which also has a hash. Each layer also references the hash of the previous layer.
4. This allows a linked list structure of layers, and all of these layers are combined into our final output image.

## Starting a container
Woohoo, you made it this far! Lucky for you, starting a container might be the easiest part of this whole thing.

All it takes is one more command:

```dockerfile

```