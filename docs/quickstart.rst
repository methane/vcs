.. _quickstart:

Quickstart
==========

Say you don't want to install ``vcs`` or just want to begin with really fast
tutorial?  Not a problem, just follow sections below.

Prepere
-------

We will try to show you how you can use ``vcs`` directly on repository. But hey,
``vcs`` is maintained within mercurial
`repository <http:http://bitbucket.org/marcinkuzminski/vcs/>`_ already, so why
not use it? Simply run following commands in your shell

.. code-block:: bash

   cd /tmp
   hg clone http://bitbucket.org/marcinkuzminski/vcs/
   hg update web # temporary, new parts haven't yet landed into default branch
   cd vcs

Now run your python interpreter of choice::

   $ python
   >>>

.. note::
   You may of course put your clone of ``vcs`` wherever you like but running
   python shell *inside* of it would allow you to use just cloned version of
   ``vcs``.

Take the shortcut
-----------------

There is no need to import everything from ``vcs`` - in fact, all you'd need is
to import ``get_repo``, at least for now. Then, simply initialize repository
object by providing it's type and path.

.. code-block:: pycon

   >>> from vcs import get_repo
   >>>
   >>> # create mercurial repository representation at current dir
   >>> # alias tells which backend should be used (see vcs.BACKENDS)
   >>> repo = get_repo(alias='hg', path='.')

Basics
------

Let's ask repo about the content...

.. code-block:: python

   >>> root = repo.request('')
   >>> print root.nodes # prints nodes of the RootNode
   [<DirNode ''>, <DirNode 'docs'>, <DirNode 'tests'>, # ... (chopped)
   >>>
   >>> # get 10th changeset
   >>> chset = repo.get_changeset(10)
   >>> print chset
   <MercurialChangeset at 10>
   >>>
   >>> # any backend would return latest changeset if revision is not given
   >>> tip = repo.get_changeset()
   >>> tip is repo.get_changeset('tip') # for mercurial backend 'tip' is allowed
   True
   >>> tip is repo.get_changeset(None) # any backend allow revision to be None (default)
   True
   >>> tip.revision is repo.revisions[-1]
   True
   >>>
   >>> # Iterate repository
   >>> list(repo) == repo.changesets.values()
   True
   
Walking
-------

Now let's ask for nodes at revision 44
(http://bitbucket.org/marcinkuzminski/vcs/src/a0eada0b9e4e/)

.. code-block:: python

   >>> root = repo.request('', 44)
   >>> print root.dirs
   [<DirNode 'docs'>, <DirNode 'examples'>, <DirNode 'tests'>, <DirNode 'vcs'>]

.. note::
   :ref:`api-nodes` are objects representing files and directories within the
   repository revision.

.. code-block:: python

   >>> # Fetch vcs directory
   >>> vcs = repo.request('vcs', 44)
   >>> print vcs.dirs
   [<DirNode 'vcs/backends'>, <DirNode 'vcs/utils'>, <DirNode 'vcs/web'>]
   >>> web_node = vcs.dirs[-1]
   >>> web = repo.request(web_node.path, 44)
   >>> print web.nodes
   [<DirNode 'vcs/web/simplevcs'>, <FileNode 'vcs/web/__init__.py'>]
   >>> print web.files
   [<FileNode 'vcs/web/__init__.py'>]
   >>> web.files[0].content
   ''
   >>> print vcs.files[0].content
   """
   Various Version Control System management abstraction layer for Python.
   """
   
   VERSION = (0, 0, 1, 'alpha')
   
   __version__ = '.'.join((str(each) for each in VERSION[:4]))
   
   __all__ = [
       'get_repo', 'get_backend', 'BACKENDS',
       'VCSError', 'RepositoryError', 'ChangesetError']
   
   from vcs.backends import get_repo, get_backend, BACKENDS
   from vcs.exceptions import VCSError, RepositoryError, ChangesetError
   

   >>> chset44 = repo.get_changeset(44)
   >>> chset44.get_node('vcs/web') is web
   True
   >>> # same if we span ``get_node`` methods:
   >>> chset44.get_node('vcs').get_node('web') is web
   True

Getting meta data
-----------------

Make ``vcs`` show us some meta information

Tags and branches
~~~~~~~~~~~~~~~~~

.. code-block:: python
   
   >>> repo.branches
   ['default', 'web']
   >>> repo.tags
   ['tip']
   >>> # get changeset we know well
   >>> chset44 = repo.get_changeset(44)
   >>> chset44.branch
   'web'
   >>> chset44.tags # most probably empty list after commit
   []

Give me a file finally!
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   >>> root = repo.request('', 44)
   >>> backends = root.get_node('vcs/backends')
   >>> backends.files
   [<FileNode 'vcs/backends/__init__.py'>,
    <FileNode 'vcs/backends/base.py'>,
    <FileNode 'vcs/backends/hg.py'>]
   >>> f = backends.get_node('hg.py')
   >>> f.name
   'hg.py'
   >>> f.path
   'vcs/backends/hg.py'
   >>> f.size
   8882
   >>> f.last_changeset
   <MercurialChangeset at 44>
   >>> f.last_changeset.date
   datetime.datetime(2010, 4, 14, 14, 8)
   >>> f.last_changeset.message
   'Cleaning up codes at base/mercurial backend'
   >>> f.last_changeset.author
   'Lukasz Balcerzak <lukasz.balcerzak@python-center.pl>'
   >>>
   >>> f.mimetype
   'text/x-python'
   >>>
   >>> # Following would raise exception unless you have pygments installed
   >>> f.lexer
   <pygments.lexers.PythonLexer>
   >>> f.lexer_alias # shortcut to get first of lexers' available aliases
   'python'
   >>> f.name
   >>>
   >>> # wanna go back? why? oh, whatever...
   >>> f.parent
   <DirNode 'vcs/backends'>
   >>>
   >>> # is it cached? hell yeah...
   >>> f is f.parent.get_node('hg.py') is repo.request('vcs/backends/hg.py', 44)
   True
