---
title: "File I/O with Containers"
teaching: 15
exercises: 5
questions:
- "How do containers interact with my local file system?"
objectives:
- "Copy files to and from the docker container"
- "Mount directories to be accessed and manipulated by the docker container"
keypoints:
- "Copy files with `docker cp`"
- "Mount volumes with `docker run -v <path on host>:<path in container> <image>`"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/t93igGemQi8?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Copying

[Copying][docker-docs-cp] files between the local host and Docker containers is possible.
On your local host, either find a file that you want to transfer to the container or create a new one. Below is the procedure
for creating a new file called io_example.txt and then copying it to the container:

~~~bash
touch io_example.txt
# If on Mac need to do: chmod a+w io_example.txt
echo "This was written on local host" > io_example.txt
docker cp io_example.txt <NAME>:/home/docker/data/
~~~
{: .source}

and then from the container check and modify it in some way

~~~bash
pwd
ls
cat io_example.txt
echo "This was written inside Docker" >> io_example.txt
~~~
{: .source}

>## Permission issues
>If you run into a `Permission denied` error, there is a simple and quick fix:
>~~~bash
>exit  # exit container
>chmod a+w io_example.txt  # add write permissions for all users
>~~~
>{: .source}
>And continue from the ``docker cp ...`` command above.
{: .callout}

~~~
/home/docker/data
io_example.txt
This was written on local host
~~~
{: .output}

and then on the local host copy the file out of the container

~~~bash
docker cp <NAME>:/home/docker/data/io_example.txt .
~~~
{: .source}

and verify if you want that the file has been modified as you wanted

~~~bash
cat io_example.txt
~~~
{: .source}

~~~
This was written on local host
This was written inside Docker
~~~
{: .output}

# Volume mounting

What is more common and arguably more useful is to [mount volumes][docker-docs-volumes] to
containers with the `-v` flag.
This allows for direct access to the host file system inside of the container and for
container processes to write directly to the host file system.

~~~bash
docker run -v <path on host>:<path in container> <image>
~~~
{: .source}

For example, to mount your current working directory (``$PWD``) on your local machine to the `data`
directory in the example container

~~~bash
docker run --rm -it -v $PWD:/home/docker/data matthewfeickert/intro-to-docker
~~~
{: .source}

From inside the container you can `ls` to see the contents of your directory on your local
machine

~~~bash
ls
~~~
{: .source}

and yet you are still inside the container

~~~bash
pwd
~~~
{: .source}

~~~
/home/docker/data
~~~
{: .output}

You can also see that any files created in this path in the container persist upon exit

~~~bash
touch created_inside.txt
exit
ls *.txt
~~~
{: .source}

>## Permission issues
>If you run into a `Permission denied` error, you can use ``sudo`` for a quick fix if you have superuser access (but this is not recommended).
{: .callout}

~~~
created_inside.txt
~~~
{: .output}

This I/O allows for Docker images to be used for specific tasks that may be difficult to
do with the tools or software installed on only the local host machine.
For example, debugging problems with software that arise on cross-platform software, or
even just having a specific version of software perform a task (e.g., using Python 2 when
    you don't want it on your machine, or using a specific release of
    [TeX Live][Tex-Live-image] when you aren't ready to update your system release).

<!--# Running Jupyter from a Docker Container-->
<!---->
<!--You can run a Jupyter server from inside of your Docker container.-->
<!--First run a container while [exposing][docker-docs-run-expose-ports] the container's-->
<!--internal port `8888` with the `-p` flag-->
<!---->
<!--~~~-->
<!--docker run --rm -it -p 8888:8888 matthewfeickert/intro-to-docker /bin/bash-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--Then [start a Jupyter server][jupyter-docs-server] with the server listening on all IPs-->
<!---->
<!--~~~-->
<!--jupyter notebook --allow-root --no-browser --ip 0.0.0.0-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--though for your convince the example container has been configured with these default-->
<!--settings so you can just run-->
<!---->
<!--~~~-->
<!--jupyter notebook-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--Finally, copy and paste the following with the generated token from the server as-->
<!--`<token>` into your web browser on your local host machine-->
<!---->
<!--~~~-->
<!--http://localhost:8888/?token=<token>-->
<!--~~~-->
<!--{: .source}-->
<!---->
<!--You now have access to Jupyter running on your Docker container.-->
<!---->
[docker-docs-cp]: https://docs.docker.com/engine/reference/commandline/cp/
[docker-docs-volumes]: https://docs.docker.com/storage/volumes/
[Tex-Live-image]: https://hub.docker.com/r/matthewfeickert/latex-docker/
[docker-docs-run-expose-ports]: https://docs.docker.com/engine/reference/run/#expose-incoming-ports
[jupyter-docs-server]: https://jupyter.readthedocs.io/en/latest/running.html#starting-the-notebook-server

{% include links.md %}
