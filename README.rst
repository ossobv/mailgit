mailgit :: Keep track of your mail user history using a Git repository
======================================================================

*Admittedly, this is by far not as useful as its DNS counterpart* `dnsgit`_,
*but it's still nice to have.*

If you use `postfix <http://www.postfix.org/>`_ with `virtual maps`_, you may
have your domains, users and aliases contained in a readable database. But does
it have history, and easy searching?

In our case, we're using an old `Modoboa <https://modoboa.org/en/>`_ database
layout, with some extra hacks to make things work for us. However, quickly
finding the current running config and keeping track of changes is hard.
Running *mailgit* as a cron job gives us a daily digest of changes and history.

**On a daily basis, it recreates a filesystem structure based on the current
domain/user/alias config and commits that state to a local Git repository.**

Any undo operation still has to be performed manually, but at least
you'll know what to undo.

*NOTE: As this is particularly dependent on your SQL database scheme, you'll
probably need to tweak* ``/etc/default/mailgit`` *heavily. It probably won't
even work without some changes to this code. Pull requests are welcome.*

.. _dnsgit: https://github.com/ossobv/dnsgit
.. _`virtual maps`: http://www.postfix.org/VIRTUAL_README.html


Example setup
-------------

.. code-block:: console

    # install -T -o755 mailgit /usr/local/bin/mailgit
    # mkdir /srv/mailgit
    # cd /srv/mailgit

    # git init; touch README.rst; git add README.rst; git commit -m Initial

    # mailgit update-all

Run that last one -- ``mailgit update-all`` -- from a daily cron job.

This will initialize and update the local repository and keep track of
changes.

For instance, a change may look like this:

.. code-block:: console

    # git show 1fb2456b71d2c884971ca23bf67ea1bd12beefa4
    commit 1fb2456b71d2c884971ca23bf67ea1bd12beefa4
    Author: OSSO <anonymous@anonymous.invalid>
    Date:   Mon May 13 17:46:44 2019 +0200
    
        changed: alias@example.com relay@internal.com
    
    diff --git a/e/example.com/alias@example.com b/e/example.com/alias@example.com
    deleted file mode 100644
    index b204fa5..0000000
    --- a/e/example.com/alias@example.com
    +++ /dev/null
    @@ -1 +0,0 @@
    -info@remote-example.com
    diff --git a/i/internal.com/relay@internal.com b/i/internal.com/relay@internal.com
    new file mode 120000
    index 0000000..c134226
    --- /dev/null
    +++ b/i/internal.com/relay@internal.com
    @@ -0,0 +1 @@
    +support@internal.com
    \ No newline at end of file

In this case an alias file (for a remote domain) was removed, and symlink was
added to denote an added alias to an existing local domain.

*I'm not sure yet if I'm fond of the symlink-for-existing-domains and
files-for-external-domains scheme. The symlinks do not make* ``git log -S``
*searching easier.*


License
-------

mailgit is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free
Software Foundation, either version 3 of the License, or (at your
option) any later version.
