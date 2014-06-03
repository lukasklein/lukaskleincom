Check if IP is in CIDR subnet
#############################

:date: 2013-08-02 14:00
:tags: django
:category: django
:slug: django-ip-cidr-check

Sometimes, you need to check if an IP address is in a specific subnet. E.g.,
when writing a `GitHub webhook endpoint`_, you want to check that the
originating IP is one of GitHub's. There's an `API call`_ you can make to get
the list of subnets hook calls can originate from:

.. code-block:: bash

	$ curl -i https://api.github.com/meta
	HTTP/1.1 200 OK
	Server: GitHub.com
	Date: Fri, 02 Aug 2013 10:31:02 GMT
	Content-Type: application/json; charset=utf-8
	Status: 200 OK
	X-RateLimit-Limit: 60
	X-RateLimit-Remaining: 59
	X-RateLimit-Reset: 1375443051
	X-GitHub-Media-Type: github.beta
	X-Content-Type-Options: nosniff
	Content-Length: 131
	Access-Control-Allow-Credentials: true
	Access-Control-Expose-Headers: ETag, Link, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes
	Access-Control-Allow-Origin: *
	ETag: "1ab8320a67d3ebed78b1999c824222c9"
	Cache-Control: max-age=0, private, must-revalidate
	Vary: Accept-Encoding

	{
	  "hooks": [
	    "204.232.175.64/27",
	    "192.30.252.0/22"
	  ],
	  "git": [
	    "207.97.227.239/32",
	    "192.30.252.0/22"
	  ]
	}


So, the two subnets are, in `CIDR format`_,


.. code-block:: text

	"204.232.175.64/27",
	"192.30.252.0/22"


But how do you check these in a Django view? First, let me show you a trick
how you can get the IP address in Django:

.. code-block:: python

	def get_client_ip(request):
	    x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
	    if x_forwarded_for:
	        ip = x_forwarded_for.split(',')[0]
	    else:
	        ip = request.META.get('REMOTE_ADDR')
	    return ip

This will not only check the `request.META` `REMOTE_ADDR`, but also parse
`HTTP_X_FORWARDED_FOR` headers. You will get something like

    87.188.16.49

To check this against a subnet, you first have to understand how the CIDR
format works. IPs are 32 bits, i.e. 4 bytes. Above you see the normal
human-readable notation. If you express the 4 bytes in binary, you get

    01010111101111000001000000110001

I wrote two Python functions to easily convert IP(v4)s from the human-readable
format to bits. First a function to convert a byte to bits (and pad it with 0s
to get 8 bits):

.. code-block:: python
    
    byte_to_bits = lambda b: bin(int(b))[2:].rjust(8, '0')

and another function that splits an IP by the dots and converts the single
bytes:

.. code-block:: python

    ip_to_bits = lambda ip: ''.join([byte_to_bits(b) for b in ip.split('.')])

Back to how CIDR works: You might have noticed the / in the GitHub IPs. This
tells us the size of the subnet in bits. If you have a */8 subnet*, that means
that the first *8 bits* define the subnet, e.g. `17.0.0.0/8` could be anything
from `17.0.0.0` to `17.255.255.255` (this was once called a class-a subnet and
is owned by `Apple`_).

In result, in order to check if an IP is inside GitHub's `204.232.175.64/27`,
we have to check if the **first 27 bits** of the originating IP address match
those of `204.232.175.64`.
Since we already have the functions to convert IPs to bits, all we have to do
now is to plug everything together:

.. code-block:: python

    from django.conf import settings

    def get_client_ip(request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip


    def test_ip_in_range(request):
        client_ip = get_client_ip(request)
        byte_to_bits = lambda b: bin(int(b))[2:].rjust(8, '0')
        ip_to_bits = lambda ip: ''.join([byte_to_bits(b) for b in ip.split('.')])
        client_ip_bits = ip_to_bits(client_ip)
        for net in settings.GITHUB_WEBHOOK_IPS:
            ip, snet = net.split('/')
            ip_bits = ip_to_bits(ip)
            if client_ip_bits[:int(snet)] == ip_bits[:int(snet)]:
                return True
        return False

And don't forget to set `GITHUB_WEBHOOK_URLS` in your `settings.py`:

.. code-block:: python

    GITHUB_WEBHOOK_IPS = ['204.232.175.64/27', '192.30.252.0/22', ]

This is probably far from perfect and lacks support for IPv6, but it works and
gives you an idea of how the internet protocol works.

.. _`GitHub webhook endpoint`: https://help.github.com/articles/post-receive-hooks
.. _`API call`: http://developer.github.com/v3/meta/
.. _`CIDR format`: https://en.wikipedia.org/wiki/CIDR
.. _`Apple`: https://en.wikipedia.org/wiki/List_of_assigned_/8_IPv4_address_blocks