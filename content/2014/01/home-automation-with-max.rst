Homeautomation with MAX! heating system
#######################################

:date: 2014-01-14 15:00
:tags: homeautomation
:category: homeautomation
:slug: home-automation-with-max

I recently bought the `Max! Cube LAN Gateway <http://www.amazon.de/gp/product/B00DUED4JM/ref=as_li_ss_tl?ie=UTF8&camp=1638&creative=19454&creativeASIN=B00DUED4JM&linkCode=as2&tag=lukaskleinc00-21>`_ and a `thermostat <http://www.amazon.de/gp/product/B005MXAB6S/ref=as_li_ss_tl?ie=UTF8&camp=1638&creative=19454&creativeASIN=B005MXAB6S&linkCode=as2&tag=lukaskleinc00-21>`_ from Amazon to extend my home automation efforts.

The installation was quite simple. Well, the hardware installation was. To setup the software you are provided with a Java application. Since I purged Java from my Mac, it was time to fire up a VM.

Great, so the Java app runs a local webserver and opens your browser. Why not. In the first step of the setup you have to provide the IP of the LAN gateway. Sure, how could any normal user know how to get it? Let's fire up nmap:

.. code-block:: bash

    lukas@lukass-mbp btde (master)  $ nmap -sP 192.168.178.0/24

    Starting Nmap 6.25 ( http://nmap.org ) at 2014-01-14 13:48 CET
    ...
    Nmap scan report for KHA0008960 (192.168.178.26)
    Host is up (0.0021s latency).
    ...
    Nmap done: 256 IP addresses (12 hosts up) scanned in 4.67 seconds


The device with hostname "KHA0008960" was the only device I did not know yet, so it had to be the gateway.

After the setup, a `not-so-nice <http://l.productgang.com/image/2X2P1R2n3h3j>`_ webinterface showed where you can control your thermostat. Well, let's do better.

But then...

.. code-block:: bash


    lukas@lukass-mbp btde (master)  $ nmap -p- 192.168.178.26

    Starting Nmap 6.25 ( http://nmap.org ) at 2014-01-14 14:12 CET
    Nmap scan report for KHA0008960 (192.168.178.26)
    Host is up (0.0031s latency).
    All 65535 scanned ports on KHA0008960 (192.168.178.26) are closed

    Nmap done: 1 IP address (1 host up) scanned in 24.47 seconds


It turns out that you first have to logout from the LAN gateway in order to open access to other devices. After that:

.. code-block:: bash

    lukas@lukass-mbp btde (master)  $ nmap -p- 192.168.178.26

    Starting Nmap 6.25 ( http://nmap.org ) at 2014-01-14 14:29 CET
    Nmap scan report for KHA0008960 (192.168.178.26)
    Host is up (0.0061s latency).
    Not shown: 65534 closed ports
    PORT      STATE SERVICE
    62910/tcp open  unknown

    Nmap done: 1 IP address (1 host up) scanned in 24.60 seconds
    lukas@lukass-mbp btde (master)  $ nc 192.168.178.26 62910
    H:KHA0008960,07449c,0112,00000000,7d1f30aa,02,31,0e010e,0e1e,03,0000
    M:00,01,VgIBAQtMdWthcycgUm9vbQkT6gEBCRPqS0hBMDAwOTU3MAZXaW5kb3cBAQ==
    C:07449c,7QdEnAASAf9LSEEwMDA4OTYwAAsABEAAAAAAAAAAAP///////////////////////////wsABEAAAAAAAAAAQf///////////////////////////2h0dHA6Ly9tYXguZXEtMy5kZTo4MC9jdWJlADAvbG9va3VwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAENFVAAACgADAAAOEENFU1QAAwACAAAcIA==
    C:0913ea,0gkT6gEBGP9LSEEwMDA5NTcwKyE9CQcYAzQM/wBESFUIRSBFIEUgRSBFIEUgRSBFIEUgRSBFIERIVQhFIEUgRSBFIEUgRSBFIEUgRSBFIEUgREhUbETMVRRFIEUgRSBFIEUgRSBFIEUgRSBESFRsRMxVFEUgRSBFIEUgRSBFIEUgRSBFIERIVGxEzFUURSBFIEUgRSBFIEUgRSBFIEUgREhUbETMVRRFIEUgRSBFIEUgRSBFIEUgRSBESFRsRMxVFEUgRSBFIEUgRSBFIEUgRSBFIA==
    L:CwkT6pwSGQAqAAAA

Thanks to some guys at the `Domoticaforum Europe <http://www.domoticaforum.eu/viewtopic.php?f=66&t=6654>`_ it's quite easy to read the response (and see that the firmware hasn't changed since 2011).


To make a long story short, I wrote a little `Python library <https://github.com/lukasklein/maxcontrol>`_ to interact with the thermostats. Feel free to improve it and open a pull request :)
