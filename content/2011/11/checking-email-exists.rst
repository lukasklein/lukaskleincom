Checking if an email address exists
###################################

:date: 2011-11-09 20:20
:tags: email
:category: coding
:slug: checking-email-exists

Today I was faced with an interesting question: How can you tell if an
email-address exists? Yep, not if it's valid (that's easy with some
regexp-magic), but if it actually exists. I came up with a simple
Python-solution.

What if we want to check, if, for example, *somefoobarstuff@gmail.com*, exists?
It's a valid email address, but it obviously doesn't belong to any account. My
solution uses SMTP, the protocol that is used to send emails.

At first we have to find the MX-record of the domain - it tells you what mail
server you have to use. I'm using the `Python DNS Library`_ to get the actual
records and smtplib_ to try to establish a connection.

I'm maybe a bit lazy, but instead of reading all the smtplib documentation I
decided to implement the `SMTP protocol`_ on my own. We basically only need to
go as far as to the RCPT TO command, to which the server replies with an 550
error if the recipient does not exist.

I'm maybe bad at describing things, but I hope you can find my code helpful for
whatever you're planning. But please be nice and don't use it for spamming :)

.. _`Python DNS Library`: http://sourceforge.net/projects/pydns/
.. _smtplib: http://docs.python.org/library/smtplib.html
.. _`SMTP protocol`: http://en.wikipedia.org/wiki/SMTP#SMTP_transport_example

.. code-block:: python

	import DNS, smtplib, socket

	def checkmail(mail):
		DNS.DiscoverNameServers()
		print "checking %s..."%(mail)
		hostname = mail[mail.find('@')+1:]
		mx_hosts = DNS.mxlookup(hostname)
		failed_mx = True
		for mx in mx_hosts:
			smtp = smtplib.SMTP()
			try:
				smtp.connect(mx[1])
				print "Stage 1 (MX lookup & connect) successful."
				failed_mx = False
				s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
				s.connect((mx[1], 25))
				s.recv(1024)
				s.send("HELO %s\n"%(mx[1]))
				s.recv(1024)
				s.send("MAIL FROM:< test@test.com>\n")
				s.recv(1024)
				s.send("RCPT TO:<%s>\n"%(mail))
				result = s.recv(1024)
				print result
				if result.find('Recipient address rejected') > 0:
					print "Failed at stage 2 (recipient does not exist)"
				else:
					print "Adress valid."
					failed_mx = False
				s.send("QUIT\n")
				break
			except smtplib.SMTPConnectError:
				continue
		if failed_mx:
			print "Failed at stage 1 (MX lookup & connect)."
		print ""
		if not failed_mx:
			return True
		return False