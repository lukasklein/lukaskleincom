Gathering information about your Facebook-event-attendees
#########################################################

:date: 2011-03-15 21:29
:tags: facebook, graph-api
:category: coding
:slug: gathering-information-facebook-event-attendees

We're currently in the planning phase for our next "Vorabifeier" (we're
celebrating our future Abitur) and we created a Facebook-event-page to keep
track of the expected guests. The reactions were huge, so we thought "Hey,
couldn't we use this system to collect some information about the attendees?".

With Facebook, the data kraken no. 1, this shouldn't be such a big problem.
Indeed, there's this nice Graph API, so I started hacking a tiny Python-script
which first gets the list of the people attending our event, then gathers some
information about each of this persons and finally does some crazy stuff with
it (well, it could do some crazy stuff, in this example it only counts the
people of full age).

Maybe you could make use of it.

So, here it is:

.. code-block:: python

	#!/usr/bin/env python

	"""
	Facebook-Event-Stalking-Tool
	"""

	import urllib2, json, re
	from types import NoneType

	access_token = "your facebook access token"
	event_id = "your event id"
	target_date = (4, 8, 2011) # the date (well, we actually could get this one from the event itself, but I thought it would be nice to be a bit more flexible)
	target_age = 18

	attendees_older = 0
	attendees_younger = 0

	event = urllib2.urlopen("https://graph.facebook.com/%s/attending?access_token=%s"%(event_id, access_token))#
	attendees_list = event.read()
	attendees_json = json.loads(attendees_list)
	total = len(attendees_json['data'])
	for attendee in attendees_json['data']:
		id = attendee['id']
		user = urllib2.urlopen("https://graph.facebook.com/%s?access_token=%s"%(id, access_token))
		user_ = user.read()
		user_json = json.loads(user_)
		birthday = user_json.get('birthday', '')
		birthday_bam = re.search('([0-9]{2})\/([0-9]{2})\/([0-9]{4})', birthday)
		old_enough = False
		if type(birthday_bam) != NoneType:
			birthday_bam = birthday_bam.groups(0)
			month = birthday_bam[0]
			day = birthday_bam[1]
			year = birthday_bam[2]
			old_enough = False
			if target_date[2]-target_age >= year:
				if target_date[1] < month:
					old_enough = True
				else:
					if target_date[1] == month:
						if target_date[0] <= day:
							old_enough = True
			else:
				old_enough = True
			if old_enough:
				attendees_older += 1
			else:
				attendees_younger += 1
		print "older: %d - younger: %d - to go: %d"%(attendees_older, attendees_younger, total)
		total -= 1