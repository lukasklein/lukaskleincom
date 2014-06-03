Check whether your iDevice has a camera built in in Phonegap
############################################################

:date: 2011-08-19 21:34
:tags: phonegap
:category: coding
:slug: check-idevice-camera-phonegap


Earlier this week I discovered Phonegap_ when I got a beta-invite to their new
`build-service`_. To those who never heard of Phonegap before: It's::

	an HTML5 app platform that allows you to author native applications with web technologies and get access to APIs and app stores

One of the awesome things about Phonegap is the extensibility with plugins
caused by the nature of open source. Even though the built-in features of
Phonegap already are very well-engineered, there are some cases in which you
have to write your own plugin in order to achieve the functionality you want.

For example, the camera-API doesn't offer the ability to check, whether the
device contains a camera. In a project I'm currently working on the user can
take a picture, but what if he's using an older iPod touch or an iPad 1? In
this case I'd like to hide the "Take a picture" button and instead only display
a "Choose from library" button, as you probably know it from other iOS apps.

.. _Phonegap: http://www.phonegap.com/
.. _`build-service`: http://build.phonegap.com/

It's very easy to test this in Objective-C:

.. code-block:: objective-c

	bool hascamera = [UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera];

But it's some extra effort to make this available in Phonegap. I'll be posting
the .h and .m files, which you have to put inside your Plugins-directory and
the .js file which you'll be importing in your HTML-page (therefore you have
to put it in the www-directory). For more information on how to install plugins
in Phonegap for iOS, see http://wiki.phonegap.com/w/page/43708792/How%20to%20Install%20a%20PhoneGap%20Plugin%20for%20iOS

Here's a quick demo of how to use it:

.. code-block:: javascript

	window.plugins.CameraAvailable.hasCamera(function(result) {
		if(result.available) {
			var buttons = ["Take Photo", "Choose From Library", "Cancel"];
		} else {
			var buttons = ["Choose From Library", "Cancel"];
		}
	});


CameraAvailable.h:

.. code-block:: objective-c

	//
	//  CameraAvailable.h
	//
	//
	//  Created by Lukas Klein on 08-19-11.
	//  MIT Licensed
	//  Copyright (c) Lukas Klein

	#import <foundation foundation.h="">
	#ifdef PHONEGAP_FRAMEWORK
	#import <phonegap pgplugin.h="">
	#else
	#import "PGPlugin.h"
	#endif

	@interface CameraAvailable : PGPlugin { }

	- (void)hasCamera:(NSArray*)arguments withDict:(NSDictionary*)options;

	@end

CameraAvailable.m:

.. code-block:: objective-c

	//
	//  CameraAvailable.m
	//
	//
	//  Created by Lukas Klein on 08-19-11.
	//  MIT Licensed
	//  Copyright (c) 2011 Lukas Klein

	#import "CameraAvailable.h"

	@interface CameraAvailable (Private)
	-(void) callbackWithFuntion:(NSString *)function withData:(NSString *)value;
	@end

	@implementation CameraAvailable

	- (void)hasCamera:(NSArray*)arguments withDict:(NSDictionary*)options
	{
	NSUInteger argc = [arguments count];

	if (argc < 1) {
	return;
	}
	bool hascamera = [UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera];

	NSString *callBackFunction = [arguments objectAtIndex:0];
	[self callbackWithFuntion:callBackFunction withData:[NSString stringWithFormat:@"{available: %@}", (hascamera ? @"true" : @"false")]];
	}

	-(void) callbackWithFuntion:(NSString *)function withData:(NSString *)value{
	if (!function || [@"" isEqualToString:function]){
	return;
	}

	NSString* jsCallBack = [NSString stringWithFormat:@"%@(%@);", function, value];
	[self writeJavascript: jsCallBack];
	}

	@end

CameraAvailable.js:

.. code-block:: objective-c

	//
	//  CameraAvailable.js
	//
	//
	//  Created by Lukas Klein on 08-19-11.
	//  MIT Licensed
	//  Copyright (c) Lukas Klein

	function CameraAvailable() {};

	CameraAvailable.prototype.hasCamera = function(result)
	{
	return PhoneGap.exec("CameraAvailable.hasCamera", GetFunctionName(result));
	}

	PhoneGap.addConstructor(function()
	{
	if(!window.plugins)
	{
	window.plugins = {};
	}
	window.plugins.CameraAvailable = new CameraAvailable();
	});