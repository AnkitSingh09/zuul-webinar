Zuul CI Webinar
===============

(09.10.2018)

Zuul page: https://zuul-ci.org/

Zuul documentation: https://zuul-ci.org/docs/zuul/

Crowdcast event page: https://www.crowdcast.io/e/zuul

Presentation slides: https://docs.google.com/presentation/d/1yNS8q77PI0ds248q0Ur6QySvhn7EdyBoXN1ocnwOwbQ

Zuul dashboard: http://zuul-webinar.codilime.com

Zuul deployment script used for the webinar:
https://github.com/jluk-codi/zuul-lab

Workshop manual
################

1. Create a change
--------------------------

Fork https://github.com/jluk-codi-org/sampleproject

Clone your fork:

``git clone ssh://git@github.com/YOUR-USER-HERE/sampleproject``

Checkout to a new branch:

``git checkout -b newbranch``

Edit the README.md file:

::

  vim README.md
  git commit -a -m "Edited README.md"
  git push origin newbranch

Create new pull request from your branch via the GitHub webui.

Go to your Pull Request page. You should see Zuul comments that tell you that the check
succeeded without running any useful jobs. Let's write our first job.

2. Create your first job
--------------------------

Go back to your cloned fork. Add a new job to the ``zuul.yaml`` file

::

  - job:
      name: myjob
      run: myjob.yaml

The ``run`` key specifies the filename of Ansible playbook that will be used to run the job.
Create the file:

::

  - hosts: all
    tasks:
      - name: This is a test task
        debug:
          msg: You should see me in the logs.
      - name: Another test task
        command: date

Finally, add the job to the check pipeline, so it will run after updating the PR. You should add this line to the ``zuul.yaml`` file:

::

     check:
       jobs:
         - noop
  +      - myjob

You're now ready to go, so let's update the PR:

::

  git add zuul.yaml myjob.yaml
  git commit -m "Add myjob to Zuul config"
  git push origin newbranch

You should see your job in the Zuul comment under your update on the PR page. Add this point, the job should finish with ``SUCCESS``.

3. Add some more projects
--------------------------

It's time to do something with second project. Add ``jluk-codi-org/anotherproject`` as a requirement for your job:

::

    - job:
        name: myjob
        run: myjob.yaml
  +     required-projects:
  +       - jluk-codi-org/anotherproject

Let's also see if the project is available in our workspace - add another task to ``myjob.yaml``:

::

      - name: Another test task
        command: date
  +   - name: List projects
  +     shell: ls src/*/*

Commit and update:

::

  git add zuul.yaml myjob.yaml
  git commit -m "Add anotherproject to myjob"
  git push origin newbranch

You should see the ``anotherproject`` added to the workspace.

4. Real testing
---------------

Let's add first meaningful test to our job - JSON linting:

::

      - name: List projects
        shell: ls src/*/*
  +   - name: Check JSON syntax in anotherproject
  +     shell: python -m json.tool *.json
  +     args:
  +       chdir: src/github.com/jluk-codi-org/anotherproject

Add, commit and push as usual. You should see the job failing on a bad JSON file in the required repo.

The JSON file in the ``anotherproject`` repo is broken. We'll fix it by creating a PR to the other repo and using.

Create a PR from a fork of ``https://github.com/jluk-codi-org/anotherproject``.

Edit your ``sampleproject`` PR description (first comment) and add this line:

::

  Depends-On: https://github.com/jluk-codi-org/anotherproject/pulls/YOUR-PR-NUMBER


Add a ``recheck`` comment to your PR. It will re-trigger Zuul jobs and the tests should now pass.

Additional info
#################

They're using Zuul (https://zuul-ci.org/users.html):

  * OpenStack (http://zuul.openstack.org)
  * WikiMedia (https://integration.wikimedia.org/zuul/ - version 2.5)
  * OpenLab (http://status.openlabtesting.org)
  * OpenContrail (http://zuulv3.opencontrail.org)
  * GoDaddy (private instance)
  * BMW (private instance)

Contact us:

  * Lukasz Lukasiewicz lukasz@codilime.com
  * Jarek Lukow jaroslaw.lukow@codilime.com

http://www.codilime.com
