# Docker from Scratch
> Written by Noah Smith for the CSXL Web Application and COMP 423: Foundations of Software Engineering.  
> *Last Updated: 12/28/2023*

## Preface
"So what's this whole Docker thing anyways, and is it even really that important?"
Fair question, and while the answer to that first part is a little complicated, the answer to the second part is definitely **YES**, it is that important!

Docker is a key technology, both in this course and out in industry, and is used heavily by many companies. Within this course, it's what powers tools including Dev Containers and Kubernetes/OpenShift, so understanding Docker is key to understanding them, too. In industry, even if you aren't using these exact technologies, you'll often find software engineers using Docker some way or another, and knowing how to use it will come in handy.

Anyways, now that you're (hopefully) convinced, let's dive in!

## Docker Basics
What is Docker? Basically put, **Docker is a tool for creating and running containers**. So then, what's a container?

Imagine the following scenario: You've just finished writing a program, and now you're ready to deploy it to a server. There's just one problem: That server's environment is totally different from yours! Let's say your program uses a specific version of Node and TypeScript, relies on some dependencies that need to be installed separately, and needs to be built in a certain way before you can run it. While your personal machine has all of these requirements satisfied, if you wanted to run this on the server, you'd need to manually go through all these steps and set it all up again.

Wouldn't it be easier if you could take a snapshot of an environment with everything you needed perfectly set, and then run that entire thing in a self-***contained*** environment on the server, or any other computer for that matter?

That's fundamentally what Docker does. In Docker jargon, the "snapshot" of the environment is called an **image** and the "self-contained environment" that runs that image is the **container**. Or, flipping those definitions, **a container is a running instance of an image** and an **image is a snapshot of an environment**. By these definitions, you can see that an image must exist first before a container ever can. So then, how can you create a docker image? I'm glad you asked.

## Images
Well, an image is just supposed to be a snapshot of your environment, right? So couldn't you create one by just freezing your computer in time and copying all of its contents over? Good idea! Unfortunately, that won't quite work - for one, that would take up way too much space. Instead, we'll want to distill only the *key aspects* of our environment that we need to run our program, and build an image from there.

Luckily, Docker has a simple tool for doing exactly this: The **Dockerfile**. A Dockerfile is just a step-by-step guide to creating an image, where you specify exactly what files to put where, what commands to run, and a few other things. It typically lives in the root directory of your project, and is simply called "Dockerfile" without the quotes.

Going back to our previous scenario, let's lay out the exact steps we'd need to execute in a brand new environment to get our program working.

 1. Install the correct version of Node. Let's say we're using version 20.
 2. Copy our program's files over.
 3. Install our dependencies.
 4. Build our program.
 5. Run it.

For the sake of this example, there exists a repository with a sample program already written linked [here](https://github.com/noahsmiths/Docker-Hello-World-Example). If you want to try Docker out and follow along, clone the repo and edit the Dockerfile according to the following steps.

Now, let's translate those instructions into an equivalent Dockerfile!

### 1. Install the correct version of Node.

[Insert stacks of images graphic here]

Now, you might be thinking: If we're building this image from the ground up, what's our starting point? What are we even building from? The answer is a **base image**! An image is just a snapshot of some environment, so what we can do is take one that already exists as a base, add on our changes, and then we'll have an image that does exactly what we need. Luckily for us, the good people over at Node made us an image conveniently named `node` that includes everything you need to run a Node program. Specifically, we want to use version 20, so we can append it to the name with a semicolon, like this `node:20`.

Now, in our Dockerfile, we use the `FROM` instruction to tell it what image to use as the base, like this:

    FROM node:20

There we go, the first line of our Dockerfile, done!

### 2. Copy our program's files over.
After specifying our base image, we must next specify where we want to make our changes. At this point, you can think of this as essentially being in a terminal that is, by default, at the root directory `/` of our base image. There are already a lot of other folders here, so lets create a new one just for our project called `/app`, and then switch into that. In a regular command line, that would look like two commands: `mkdir /app` to make the directory, and `cd /app` to navigate into the directory. In a Dockerfile, it's combined into one instruction called `WORKDIR`, so we would add the following line:

    WORKDIR /app

Next up, we need to copy the files over themselves. To do this, we use the `COPY` instruction, with argument