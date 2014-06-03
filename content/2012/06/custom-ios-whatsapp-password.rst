Setting a custom iOS WhatsApp password
######################################

:date: 2012-06-20 21:00
:tags: whatsapp, hack
:category: coding
:slug: custom-ios-whatsapp-password

I'm currently developing on the WhatsAPI, but there's one problem when you're
using an iPhone: no one has figured out yet how your password is being
generated. On Android devices it's just the md5 hash of the reverse of the IMEI
number of your phone, but neither this nor all possible hash-combinations of
the iPhone's UDID did word. So I did some network sniffing and figured out how
a device is being registered and how you can use this to manually reset your
WhatsApp password.

To register a device WhatsApp first sends an API request to their servers
requesting a validation code which you will either get by txt or via a call.
This request looks like::

    https://r.whatsapp.net/v1/code.php?cc=49&in=1234567890&to=00491234567890&lc=DE&lg=de&mcc=000&mnc=000&imsi=00000000000000&method=sms

where cc is the area code without leading zeros, in is the number without
country code and to is the number the sms should go to (It didn't work for me
if this was different from the one you're trying to register). The other stuff
is irrelevant (language code, don't know if/how this changes things since the
txt you'll get isn't really localized) and some other parameters you can just
set to 0.

After receiving the code you have to push out another request that is doing the
registration itself. This is the part where you can set your own password. As
I mentioned below WhatsApp uses some md5 hashes for the passwords so to not
draw attention you should do the same::

    https://r.whatsapp.net/v1/register.php?cc=49&in=12345678900&udid=05c12a287334386c94131ab8aa00d08a&code=123

Where the cc and in are the same as previously, udid is your custom password
you want to use and code is the code you just received.

From now on you can use your custom WhatsApp password. Be aware that this
method will log you out from your iPhone, you still will get push notifications
but when you open WhatsApp you will be asked to activate your device. (And, if
you do so, your password will be reset again).

To make your life even easier I've written two little and very dirty Python
functions since you need a custom user agent (I'm using the Nokia one):

.. code-block:: python

    import urllib2

    # number with 00 and countrycode, e.g. 00491234567890
    def get_new_code(number):
        uagent = "WhatsApp/2.6.10 S40Version/04.60 Device/nokiac3-00"
        url = "https://r.whatsapp.net/v1/code.php?cc=%s&in=%s&to=%s&lc=DE&lg=de&mcc=000&mnc=000&imsi=00000000000000&method=sms"%(number[2:][:2], number[4:], number)
        opener = urllib2.build_opener(urllib2.HTTPRedirectHandler())
        opener.addheaders = [('User-agent', uagent)]
        connection = opener.open(url)
        response = connection.read()
        connection.close()
        print response

    # number with 00 and countrycode like above
    # password should be a 32 character md5 lookalike to not draw attention
    # code is the 3 digit code you got in your txt
    def register_number_with_password(number, password, code):
        url = "https://r.whatsapp.net/v1/register.php?cc=%s&in=%s&udid=%s&code=%s"%(number[2:][:2], number[4:], password, code)
        opener = urllib2.build_opener(urllib2.HTTPRedirectHandler())
        opener.addheaders = [('User-agent', uagent)]
        connection = opener.open(url)
        response = connection.read()
        connection.close()
        print response
