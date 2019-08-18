======
Docker
======

Basic commands
--------------

.. code-block::

    # list images
    $ docker images

    # list containers
    $ docker ps -a

    # pull image from docker hub
    $ docker pull <image_name>

    # run container based one selected image
    $ docker run <image_name>

    # start container
    $ docker start <container_id>

    # stop container
    $ docker stop <container_id>

    # enter container's shell
    $ docker exec -it <container_id> /bin/bash

    # run container based on selected image & enter its shell
    $ docker run -it <image_name> /bin/bash

    # add npm to container based on ubuntu image
    $ apt-get update
    $ apt-get install npm
    # create new image from updated container
    $ docker commit ubuntu-npm