Mc Donalds WiFi, surrender!
###########################

:date: 2011-09-27 21:37
:tags: hack
:category: coding
:slug: mc-donalds-wifi-surrender


Ever since I'm here in New Zealand I've been using the `free WiFi`_ at every Mc
Donalds. Great! Free WiFi! =)

.. _`free WiFi`: http://www.mcdonalds.com/us/en/services/free_wifi.html

But wait, what's the catch? Well, it's limited to 50MB data per day. Uhm.
Sounds like a challenge, doesn't it? ;)

So, how does this work? They're probably identifying you by your Mac address.
To test this I used all my data allocation, changed my Mac address via::

	sudo ifconfig en0 ether <yournewrandomaddress>

and ta-da, another 50MB! But wouldn't it be great to automate this process?
It is! (Note: I'm on a Mac, but for your Linux computer it should work similar,
if you proofed it please leave a comment below :))

What the script needs to do is:

1. Set a new Mac address
2. Disconnect from the WiFi
3. Reconnect
4. Sounds like a challenge? Well, not really.


.. code-block:: python

	rootpass = getpass.getpass()
	interface = 'en0'
	def setMAC(mac):
		os.system("echo '%s' | sudo -S -s ifconfig %s ether %s"%(rootpass, interface, mac))

	def disconnect():
		os.system("networksetup -setairportpower %s off"%(interface))

	def connect():
		os.system("networksetup -setairportpower %s on"%(interface))


To generate a random Mac address I use the following function:

.. code-block:: python

	def randomMAC():
		mac = [ 0x00, 0x16, 0x3e,
			random.randint(0x00, 0x7f),
			random.randint(0x00, 0xff),
			random.randint(0x00, 0xff) ]
		return ':'.join(map(lambda x: "%02x" % x, mac))


Alright, so now we got everything we need to reconnect to the WiFi. But wait,
what about this fancy login-page, where you have to confirm the ToS before you
can use the internet? In fact, that was more like a challenge. Let's fire
up Wireshark and try to connect to google.com. What happens: You'll be
redirected to a perlscript on a local server::

	http://192.168.59.35/login.pl?action=which_interface&destination;=http://www.google.com/%3f

What's next? Believe it or not, but what follows is another redirect.::

	http://login1.maccasfreewifi.net.nz/login.php?controller=192.168.59.35&source;=192.168.38.131&destination;=http%3A%2F%2Fwww.google.com%2F%3F≈_name=&ssid;=≈=&mac;=00%3A16%3A3e%3A7e%3Ab1%3A7f

This is finally the loginpage you get to see in your browser. It contains a
form with some fancy hidden inputs like

.. code-block:: html

	<input type="hidden" name="bs_password" value="bigmac" />

McDonalds? I love you for that! :D

In fact there are these fields:

* bs_name
* bs_password
* username1
* password1
* which_form
* destination
* agree (a checkbox)

Now we could just hardcode all the parameters into our script or we could do
it the fun way:

.. code-block:: python

	f = urllib2.urlopen(loginurl)
	html = f.read()
	for line in html.split("\n"):
		if line.find('bs_name') > -1:
			bs_name = line[line.find('ue="')+4:line.find('">')]
		elif line.find('bs_password') > -1:
			bs_password = line[line.find('ue="')+4:line.find('">')]
		elif line.find('username1') > -1:
			username1 = line[line.find('ue="')+4:line.find('">')]
		elif line.find('password1') > -1:
			password1 = line[line.find('ue="')+4:line.find('">')]
		elif line.find('which_form') > -1:
			which_form = line[line.find('ue="')+4:line.find('">')]
		elif line.find('destination') > -1:
			destination  = urllib.quote(line[line.find('ue="')+4:line.find('">')])
		elif line.find('method="POST"') > -1:
			action = line[line.find('on="')+4:line.find('"" acc')]
	data = "bs_name=%s&bs;_password=%s&username1;=%s&password1;=%s&which;_form=%s&destination;=%s&agree;=1&submitButton;=&submitButton.x;=1&submitButton.y;=2"%(bs_name, bs_password, username1, password1, which_form, urllib.quote_plus(testurl))
	req = urllib2.Request(url=action, data=data)
	o = urlparse(loginurl)
	req.add_header('Content-Type', 'application/x-www-form-urlencoded')
	req.add_header('Origin', "%s://%s"%(o.scheme, o.netloc))
	req.add_header('Referer', loginurl)
	req.add_header('User-Agent', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.186 Safari/535.1')
	f = urllib2.urlopen(req)

Now we can put it all together into one simple script that checks periodically whether the connection is still alive and if not, automagically reconnects us:

.. code-block:: python

	import random, os, urllib, urllib2, time, getpass
	from urlparse import urlparse

	rootpass = getpass.getpass()
	interface = "en0"
	testurl = "http://www.google.com/robots.txt"
	stringtotest = "User-agent: *"

	def randomMAC():
		mac = [ 0x00, 0x16, 0x3e,
			random.randint(0x00, 0x7f),
			random.randint(0x00, 0xff),
			random.randint(0x00, 0xff) ]
		return ':'.join(map(lambda x: "%02x" % x, mac))
		
	def setMAC(mac):
		os.system("echo '%s' | sudo -S -s ifconfig en0 ether %s"%(rootpass, mac))

	def disconnect():
		os.system("networksetup -setairportpower %s off"%(interface))

	def connect():
		os.system("networksetup -setairportpower %s on"%(interface))
		
	def testconnection():
		f = urllib2.urlopen(testurl)
		return stringtotest==f.read(len(stringtotest))

	def login():
		print "Logging in..."
		# first stage
		request = urllib2.Request(testurl)
		opener = urllib2.build_opener()
		f = opener.open(request)
		newurl = f.url
		request = urllib2.Request(newurl)
		f = opener.open(request)
		loginurl = f.url

		f = urllib2.urlopen(loginurl)
		html = f.read()
		for line in html.split("\n"):
			if line.find('bs_name') > -1:
				bs_name = line[line.find('ue="')+4:line.find('">')]
			elif line.find('bs_password') > -1:
				bs_password = line[line.find('ue="')+4:line.find('">')]
			elif line.find('username1') > -1:
				username1 = line[line.find('ue="')+4:line.find('">')]
			elif line.find('password1') > -1:
				password1 = line[line.find('ue="')+4:line.find('">')]
			elif line.find('which_form') > -1:
				which_form = line[line.find('ue="')+4:line.find('">')]
			elif line.find('destination') > -1:
				destination  = urllib.quote(line[line.find('ue="')+4:line.find('">')])
			elif line.find('method="POST"') > -1:
				action = line[line.find('on="')+4:line.find('"" acc')]
		data = "bs_name=%s&bs;_password=%s&username1;=%s&password1;=%s&which;_form=%s&destination;=%s&agree;=1&submitButton;=&submitButton.x;=1&submitButton.y;=2"%(bs_name, bs_password, username1, password1, which_form, urllib.quote_plus(testurl))
		req = urllib2.Request(url=action, data=data)
		o = urlparse(loginurl)
		req.add_header('Content-Type', 'application/x-www-form-urlencoded')
		req.add_header('Origin', "%s://%s"%(o.scheme, o.netloc))
		req.add_header('Referer', loginurl)
		req.add_header('User-Agent', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.186 Safari/535.1')
		f = urllib2.urlopen(req)
		print "Success! :)"
				

	def reconnect():
		setMAC(randomMAC())
		disconnect()
		connect()

	if __name__ == "__main__":
		while 1:
			if not testconnection():
				print "data exceeded, reconnecting..."
				reconnect()
				time.sleep(10)
				login()
				
			time.sleep(1)

It's not perfect, but at least it works! :)