==============
git cheatsheet
==============

Git is a version-control system for tracking changes in computer files and coordinating work on those files among multiple people. It is primarily used for source-code management in software development, but it can be used to keep track of changes in any set of files.

Initialize git
--------------

.. code-block::

    # initialize git in your project’s main folder
    $ git init

Git workflow
------------

.. figure:: /img/git_workflow.png

Basic
-----

.. code-block::

    # check new files, modified files, deleted files,... in working directory
    $ git status

    # add files to the staging
    $ git add .

    # add single files to the staging
    $ git add <file_name_1> <file_name_2>

    # commit staged files to currently active branch in local .git
    $ git commit m "message"

    # check log of a currently active branch
    $ git log

    # check short log of a currently active branch
    $ git log --oneline

    # push your changes to the remote
    $ git push origin <branch_name>

Branches
--------

.. code-block::

    # show branches and currently active branch
    $ git branch

    # show local and remote branches with latest commits
    $ git branch -va

    # create new branch
    $ git branch <branch_name>

    # activate branch and work on it
    $ git checkout <branch_name>

    # create and activate branch in single command
    $ git checkout -b <branch_name>

    # merge branch to the currently active branch
    $ git merge <branch_name>

Remote
------

.. code-block::

    # connect your local project to the remote project
    $ git remote add origin <project_url>

    # push your local changes to the remote master branch
    $ git push origin master

    # push your local changes on specific branch to the same branch on remote
    $ git push origin <branch_name>

    # force push to specific branch on remote
    $ git push origin <branch_name> --force

    # pull changes from the branch on remote
    $ git pull origin <branch_name>

    # clone remote project to your computer
    $ git clone <project_url>

Collaboration
-------------

.. code-block::

    # check changes on the remote
    $ git remote show origin

    # fetch changes from remote to local
    $ git fetch origin

    # merge fetched changes from remote branch to your local branch
    $ git checkout <branch_name>

Not yet classified
------------------

.. code-block::

    $ git rebase <branch_name>

    # uncommit last X commits and commit them in one new commit
    $ git reset --soft HEAD~X
    $ git status
    $ git commit -m "message"

    # revert to the commit with short commit SHA
    $ git reset --hard <short_commit_sha>

    # copy a commit with short commit SHA from other branch and paste it to currently active branch
    $ git cherry-pick <short_commit_sha>

    # save changes
    $ git stash
