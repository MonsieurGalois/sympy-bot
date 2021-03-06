SymPy Bot
=========

The goal of SymPy Bot is to do all the automated testing for a pull request and
report back into the pull request with the results.

So far one has to run the bot manually, but eventually we would like to create
a web service for it.

Usage
-----

List all pull requests, sorted by date::

    ./sympy-bot list

Make an automatic review of a pull request::

    ./sympy-bot review 268

This will run all tests and then comment in the pull request (under your name)
with the results.

To review all open pull requests, do::

    ./sympy-bot review all

to only review mergeable pull requests, do::

    ./sympy-bot review mergeable

Requirements
------------

SymPy bot needs argparse to run. This is part of the standard library in
Python 2.7, however it can be installed in earlier versions of Python.

Tips
----

By default, the SymPy repository is fully downloaded from the web, so you don't
need to have any local copy. However, if you do have a local copy already, you
can skip most of the download (which might take a few minutes on slower
connections) by passing a ``--reference`` option to sympy-bot::

    ./sympy-bot review 268 --reference ~/repo/git/sympy

This gets passed to git, see ``git clone --help`` for more information. Then
sympy-bot starts testing the branch immediately, even if you have a slower
connections.

Configuration
-------------

You can configure SymPy bot to remember your GitHub credentials, use an
existing clone of sympy and run interpreters under different profiles. This is
done in the ``~/.sympy/sympy-bot.conf`` file. The configuration supports
multiple profiles, but will always read in the [DEFAULT] profile, so you should
start your configuration with your GitHub credentials in the default profile::

    [DEFAULT]
    user = username

The first time you run sympy-bot and need to authenticate with GitHub, you will
be prompted to create a configuration file and OAuth token. Your username and
token will automatically be put into your default profile. To add token
information on your own, you can add the following line::

    token = your GitHub API token

You can create this token from the command line by running::

    curl -u 'username' -d '{"scopes":["public_repo"],"note":"SymPy Bot"}' \
    https://api.github.com/authorizations

and enter your password at the prompt. In the information that is printed,
taken the token and add it to your configuration file, as above.

If you intend on publishing you configuration files publicly, you can put your
token in a separate file and reference that file in your configuration like
this::

    token_file = ~/path/to/token

The token file is a text file containting only your token on a single line.

You can manage your authorizations at https://github.com/settings/applications.
If you use token authentication, for your security, be sure that the
configuration file and token have the proper permissions assigned, e.g. 600. If
you supply a username without a password or API token, then sympy-bot will ask
you for your GitHub password on each invocation.

If you have an existing clone of sympy, you can avoid having to clone the SymPy
repository every time the bot is run::

    reference = ~/path/to/sympy

You can specify the interpreters to use by giving a comma separated list of
Python interpreters::

    interpreter = /path/to/python, /path/to/other/python
    interpreter3 = /path/to/python3, /path/to/other/python3

which sets the Python 2 and Python 3 interpreters, respectively. By default,
the interpreters set by ``interpreter`` are run. If you pass the ``-3`` flag or
set ``python3 = True`` in the configuration file, then ``interpreter3`` will be
used instead. You can force both by passing ``-2`` and ``-3`` or setting both
``python2 = True`` and ``python3 = True`` in the configuration file. Setting
``interpreter = None`` will disable the Python tests, which can be useful in
setting up a profile just for testing docs.

If you want to test the building of the HTML docs, you can use the ``-D`` flag
or set ``build_docs = True`` in the configuration file. By default, this will
disable running the tests. This can be overridden by setting ``python2`` or
``python3`` options, as above.

Any of the other options set by commandline parameters can be set in the
configuration file. See ``sympy-bot review --help`` for more information (the
configuration values are the long form of the option, with any dashes replaced
with underscores, for example, ``--build-docs`` becomes ``build_docs``).

The configuration also supports different profiles. To set these up, you put
the name of the profile between square brackets. Then, when you pass
``--profile profile_name``, the options in the specified section will override
the default section. This is done in the config file::

    [profile_name]
    interpreter = /path/to/different/python
    testcommand = bin/test --other-options

This can be useful for setting up various suites of tests, e.g. slow tests,
32-bit/64-bit tests, etc.

To see an example configuration file, see the ``sympy-bot.conf.example``
file.  This file also explains how you can use variable interpolation to avoid
duplication.

Foreign repositories
--------------------

SymPy Bot can be also used with other remote repository than sympy/sympy.
You can change the remote with ``-R`` flag to sympy-bot or by setting
``repository`` in configuration file. The new remote doesn't have to be
SymPy's repository, but any repository on GitHub. Note that in this case
you may need to setup customized ``testcommand``.

Custom Master Commit
--------------------

By default, sympy-bot merges with master before testing, failing if the
merge fails.  You can customize this behavior with the ``-m`` option to
``sympy-bot``.  Pass any valid git commit name to this option, and it
will use it to merge the master branch.  The default is
``origin/master``, which is the current master.  If you don't want to
merge at all, pass ``HEAD``, which will perform a noop merge against the
branch you are testing.

If you use ``--reference``, git will pull in all commits from the local
repository. Thus, you can merge with commits that are not in the
official ``sympy/sympy`` repository by using this and passing the SHA1
of the commit you want.

This is also useful for bisecting problems with SymPy Bot. Simply use
git to bisect in your local SymPy repository and pass the SHA1's it
picks to ``sympy-bot -n -m``.

Web interface integration with github
-------------------------------------

This way is a bit complicated in set up than previous (poll github for new pulls),
but that will update information about pulls in real time.

SymPy Bot web-interface (which located in under web/) supports integration with
github via mechanism called hooks http://developer.github.com/v3/repos/hooks/

To use that feature you need to follow these steps:

1. Go to ``http://example.com/upload_pull``, sign in as administrator and press
   ``generate`` button. After that, all admins will recieve notification with
   secret URL (you can see a log of all generations in table on that page)
2. You need to tell github to use this URL, so here steps (replace ``username``
   and ``repo`` with you values):
        - Go to https://github.com/user/repo/admin/hooks
        - Click on ``WebHook URLs`` and add secret URL there.
        - Find the hook that you want to modify by::

            curl -u username https://api.github.com/repos/username/repo/hooks

          the ``id`` field gives the hook ID, copy and paste the path in the
          "url" field into the command::

            curl -u username -d '{ "events": [ "pull_request" ] }'
            https://api.github.com/repos/username/repo/hooks/ID

          You will see that the "events" part::

            "events": [
                "push"
            ],

          changed to::

            "events": [
                "pull_request"
            ],
