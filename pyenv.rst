=====
pyenv
=====

pyenv lets you easily switch between multiple versions of Python. It's simple, unobtrusive, and follows the UNIX tradition of single-purpose tools that do one thing well.

Installation
------------

.. code-block::

    # Install pyenv
    brew install pyenv

    # Add pyenv initializer to shell startup script
    echo 'eval "$(pyenv init -)"' >> ~/.bash_profile

    # Reload your profile
    source ~/.bash_profile

.. code-block::

    # Install pyenv-virtualenv
    brew install pyenv-virtualenv

    # Add pyenv-virtualenv initializer to shell startup script
    echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile

    # Reload your profile
    source ~/.bash_profile

Usage
-----

.. code-block::

    # Install specific python version
    $ pyenv install 3.5.2

    # List installed python versions
    $ pyenv versions

    # Create virtulenv with specific python version
    $ pyenv virtualenv <python-version> <name>

    # Activate virtualenv
    $ pyenv activate <name>

    # Deactivate virtualenv
    $ pyenv deactivate

    # Delete virtualenv
    $ pyenv uninstall <name>