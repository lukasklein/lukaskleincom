Changing request.POST values in Django
######################################

:date: 2012-10-26 12:00
:tags: django
:category: django
:slug: changing-request-post-django

When you try to change request.POST in a Django view it will raise a QueryDict
instance is immutable exception. In order to change/add values you have to
create a shallow copy instead of a binding.

.. code-block:: python

	def index(request):
	    if request.method == 'POST':
	        post_mutable = request.post.copy()
	        # Now you can change values:
	        post_mutable['answer'] = 42