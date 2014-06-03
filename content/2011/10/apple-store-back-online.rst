Is the Apple Store back online?
###############################

:date: 2011-10-19 21:44
:tags: apple
:category: coding
:slug: apple-store-back-online


Wow, the Apple store is down! Well, it's Tuesday afternoon here in New Zealand and no one really expects anything more than, say, maintenance. But anyway, I caught myself reloading the page every few seconds. Hm, there has to be an easier way.

It is :)

.. code-block:: python

	#!/usr/bin/env python
	import urllib, time, os

	storeurl = 'http://store.apple.com/'
	needle = 'http://images.apple.com/r/store/backsoon/title_backsoon1.gif'

	def check_online():
		f = urllib.urlopen(storeurl)
		if needle in f:
			success = 'Yey, the Apple store is back online! Go check the new stuff!'
			print success
			os.system('say "%s"'%(success))
			exit()
		else:
			print 'Still offline. :('
		f.close()

	while True:
		check_online()
		time.sleep(10)