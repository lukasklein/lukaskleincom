Hack.lu CTF 2013: Challenge 7 "Robots Exclusion Committee" (web)
################################################################

:date: 2013-10-24 10:00
:tags: ctf, hack.lu, writeup
:category: ctf
:slug: hacklu13-ctf-7-robotsexclusioncommittee

The challenge was to find the *first* secret from a given webpage. The problem was that there was only a form that didn't really work (it was meant to send you an email with login credentials).

Since this was apparently not doing anything I didn't spend too much time on it and went on and tried to find the login URL.
Where could you probalby find it? Well, since the theme of the site was robots, my first guess was the ``robots.txt`` and there was it:

    # Keep em' away
    User-agent: *
    Disallow: /vault

`/vault` gave me a

    WWW-Authenticate: Basic realm="Controller"

Obvious combinations like ``admin``/``password`` didn't work ;) so I tried something else.
From my previous experience I remembered that some security-aware admins configured their server with

    <Limit GET POST>

, so the next thing I tried was a ``GETS`` to the url. Unfortunately the allowed methods were only ``GET``, ``HEAD`` and ``OPTIONS``.

Finally, `Endres`_ found out that it was vulnerable to SQL injections. Who would've known :)

The first obvious step was to login as admin with ``' OR ''='``. Unfortunately, the only secret shown was #2. Bummer. The good thing though was that the site displayed the username of the 'logged in' user. Apparently they were showing the value of row 1 in column 1, so we had a nice information disclosure vector.
A quick lookup of the users table showed us that there were 3 other users, but they showed the same secret. Trying to access ``INFORMATION_SCHEMA`` yielded in errors, so we became suspicious if MySQL was used at all.
Our next guess was SQLite, and there we go,

.. code-block:: sql

    ' UNION SELECT tbl_name FROM sqlite_master WHERE tbl_name LIKE '%secret%'; --

showed us the table name of ``hiddensecrets``.

What could be the scheme of the table?

.. code-block:: sql

    ' UNION SELECT sql FROM sqlite_master WHERE tbl_name='hiddensecrets'; --

yielded in

.. code-block:: sql

    CREATE TABLE hiddensecrets (id INTEGER PRIMARY KEY AUTOINCREMENT, val TEXT)

and a quick query for ``id`` 1 gave us the base64-encoded captcha-like image which contained the flag :)

.. _`Endres`: http://www.e7p.de/