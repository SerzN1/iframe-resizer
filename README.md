# iFrame Resizer [![Code Climate](https://codeclimate.com/github/davidjbradshaw/iframe-resizer.png)](https://codeclimate.com/github/davidjbradshaw/iframe-resizer) [![Build Status](https://travis-ci.org/davidjbradshaw/iframe-resizer.png?branch=master)](https://travis-ci.org/davidjbradshaw/iframe-resizer)

This library enables the automatic resizing of the height and width of both same and cross domain iFrames to fit the contained content. It uses [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/window.postMessage) to pass messages between the host page and the iFrame and when available [MutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver) to detect DOM changes, with a fall back to setInterval for IE8-10. 

The code also detects resize events, provides functions to allow the iFrame to set a custom size, close itself and send simple messages to the parent page. For security the hostpage automatically checks the origin of incoming messages are from the domain of the page listed in the `src` property of the iFrame.

The package contains two minified JavaScript files in the [js](js) folder. The first ([iframeResizer.min.js](https://raw2.github.com/davidjbradshaw/iframe-resizer/master/js/iframeResizer.min.js)) is for the page hosting the iFrames. It can be called with **native** JavaScript;

```js
iFrameResize([{options}],[selector]);
```

or via **jQuery**. (See [notes](https://github.com/davidjbradshaw/iframe-resizer#browser-compatibility) below for using native version with IE8).

```js
$('iframe').iFrameResize([{options}]);
```

The second file ([iframeResizer.contentWindow.min.js](https://raw.github.com/davidjbradshaw/iframe-resizer/master/js/iframeResizer.contentWindow.min.js)) is a **native** JavaScript file that needs placing in the page contained within your iFrame. <i>This file is designed to be a guest on someone else's system, so has no dependancies and won't do anything until it's activated by a message from the containing page</i>.

### Typical setup

The normal configuration is to have the iFrame resize when the browser window changes size or the content of the iFrame changes. To set this up you need to configure one of the dimensions of the iFrame to a percentage and tell the library only to update the other dimension. Normally you would set the width to 100% and have the height scale to fit the content.

```html
<iframe src="http://anotherdomain.com/frame.content.html" width="100%" scrolling="no"></iframe>
```

Note that scrolling is set to 'no', as older versions of IE don't allow this to be turned off in code and can just slightly add a bit of extra space to the bottom of the content that it doesn't report when it returns the height.

###Example
To see this working take a look at this [example](http://davidjbradshaw.com/iframe-resizer/example/) and watch the console log.


## Options

### log

	default: false
	type: boolean

Setting the `log` option to true will make the scripts in both the host page and the iFrame output everything they do to the JavaScript console so you can see the communication between the two scripts.

### autoResize

	default: true
	type: boolean

When enabled changes to the Window size or the DOM will cause the iFrame to resize to the new content size. Disable if using size method with custom dimensions.

### bodyMargin

	default: null
	type: string || number

Override the default body margin style in the iFrame. A string can be any valid value for the CSS margin attribute, for example '8px 3em'. A number value is converted into px.

### checkOrigin

	default: true
	type: boolean

When set to true, only allow incoming messages from the domain listed in the `src` property of the iFrame tag. If your iFrame navigates between different domains, ports or protocols; then you will need to disable this option.
 
### enablePublicMethods  

	default: false
	type: boolean

If enabled, a `window.parentIFrame` object is created in the iFrame. This contains `size()`, `getId()`, `sendMessage()` and `close()` methods.

### interval

	default: 32  (in ms)
	type: number

In browsers that don't support [mutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver), such as IE10, the library falls back to using setInterval, to check for changes to the page size. The default value is equal to two frame refreshes at 60Hz, setting this to a higher value will make screen redraws noticeable to the user.

Setting this property to a negative number will force the interval check to run instead of [mutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver).

Set to zero to disable.

### heightCalculationMethod

	default: 'offset'
	values: 'offset' | 'scroll'

By default the heigh of the iFrame is calculated by converting the margin of the body to px and then adding the top and bottom figures to the height of the body tag. Setting this value to **scroll** will change this to instead use `document.documentElement.scrollHeight` to calculate the height; this can fix some problems, but comes with the side-effects of slightly over reporting the height and also DOM changes that reduce the size of the content won't always be reported, so the iFrame may not shrink with the content.

### messageCallback

	type: function ({iframe,message})

Receive message posted from iFrame with the `window.parentIFrame.sendMessage()` method.

### resizedCallback

	type: function ({iframe,height,width,type})
	
Function called after iFrame resized. Passes in messageData object containing the **iFrame**, **height**, **width** and the **type** of event that triggered the iFrame to resize.

### scrolling

    default: false
    type: boolean

Enable scroll bars in iFrame.

### sizeHeight

	default: true
	type: boolean

Resize iFrame to content height.

### sizeWidth

	default: false
	type: boolean

Resize iFrame to content width.


## IFrame Methods

To enable these methods you must set `enablePublicMethods` to `true`. This creates the `window.parentIFrame` object in the iFrame. These method should be contained by a test for the `window.parentIFrame` object, in case the page is not loaded inside an iFrame. For example:

```js
if ('parentIFrame' in window) {
	parentIFrame.close();
}
```

### parentIFrame.close()

Remove the iFrame from the parent page. 

### parentIFrame.getId()

Returns the ID of the iFrame that the page is contained in.

### parentIFrame.sendMessage(message)

Send string to containing page. The message is delivered to the `messageCallback` function.

### parentIFrame.size ([customHeight],[ customWidth])

Manually force iFrame to resize. This method optionally accepts two arguments: **customHeight** & **customWidth**. To use them you need first to disable the `autoResize` option to prevent auto resizing and enable the `sizeWidth` option if you wish to set the width. 

```js
iFrameResize({
	autoResize: false,
	enablePublicMethods: true,
	sizeWidth: true
});
```

Then you can call the `size` method with dimensions:

```js
if ('parentIFrame' in window) {
	parentIFrame.size(100); // Set height to 100px
}
```


##Troubleshooting
It is possible to have CSS that causes the content to overflow the body tag, this will prevent the iFrame being correctly sized. If this is the case the simplest fix is to add a clearfix div just before the close body tag.

```html
<div style="clear:both;"></div>
```

Alternatively you can set the `heightCalculationMethod` option to **scroll**. This will change how the iFrame calculates its height; however, it does have some side effects that are discussed in the options section. 


## Browser compatibility 
###jQuery version

Works with all browsers which support [window.postMessage](http://caniuse.com/#feat=x-doc-messaging) (IE8+).

###Native version

Additionally requires support for [Array.prototype.forEach](http://kangax.github.io/es5-compat-table/#Array.prototype.forEach) (IE9+) and [document.querySelectorAll](https://developer.mozilla.org/en-US/docs/Web/API/Document.querySelectorAll) (IE8 Standards Mode). For **IE8** force [Standards Mode](http://en.wikipedia.org/wiki/Internet_Explorer_8#Standards_mode),

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

and use the [MDN PolyFil](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) on the host page.

```js
if (!Array.prototype.forEach){
	Array.prototype.forEach = function(fun /*, thisArg */){
		"use strict";
		if (this === void 0 || this === null || typeof fun !== "function") throw new TypeError();

		var
			t = Object(this),
			len = t.length >>> 0,
			thisArg = arguments.length >= 2 ? arguments[1] : void 0;

		for (var i = 0; i < len; i++)
			if (i in t)
				fun.call(thisArg, t[i], i, t);
	};
}
```


## Install via Package Management Systems

This library can be installed via the [Bower](http://bower.io),

    bower install iframe-resizer

and [Component](http://component.io) front-end package management systems.

    component install davidjbradshaw/iframe-resizer


##Changes between version 1 and 2.

Version 2 makes a few changes that you need to be aware of when upgrading. The filename of the host page script has been renamed from jquery.iframeResizer.min.js to iframeResizer.min.js in order to reflect that jQuery is now an optional way of calling the script. The do(Heigh/Width) options have been renamed size(Height/Width), contentWindowBodyMargin has been renamed bodyMargin and callback has been renamed resizedCallback.

The method names deprecated in version 1.3.0 have now been removed. Versions 1 and 2 remain compatable with each other so you can use version 2 of the hostpage script with an iFrame running version 1 of the iFrame script, or *vice versa*, however, it should be noted that the V1 iFrame script only accepts number values for bodyMargin.


##Version History
* v2.1.1 Option to change the height calculation method in the iFrame from offsetHeight to scrollHeight. Troubleshooting section added to docs.
* v2.1.0 Added sendMessage() and getId() to window.parentIFrame. Changed width calculation to use scrollWidth. Removed deprecated object name in iFrame.
* v2.0.0 Added native JS public function, renamed script filename to reflect that jQuery is now optional. Renamed *do(Heigh/Width)* to *size(Height/Width)*, renamed *contentWindowBodyMargin* to *bodyMargin* and renamed *callback* *resizedCallback*. Improved logging messages. Stop *resize* event firing for 50ms after *interval* event. Added multiple page example. Workout unsized margins inside the iFrame. The *bodyMargin* propety now accepts any valid value for a CSS margin. Check message origin is iFrame. Removed deprecated methods.
* v1.4.4 Fixed *bodyMargin* bug.
* v1.4.3 CodeCoverage fixes. Documentation improvements.
* v1.4.2 Fixed size(250) example in IE8.
* v1.4.1 Setting `interval` to a negative number now forces the interval test to run instead of [MutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver).
* v1.4.0 Option to enable scrolling in iFrame, off by default. Bower dependancies updated.
* v1.3.7 Stop *resize* event firing for 50ms after *size* event. Added size(250) to example.
* v1.3.6 Updated jQuery to v1.11.0 in example due to IE11 having issues with jQuery v1.10.1.
* v1.3.5 Documentation improvements. Added Grunt-Bump to build script.
* v1.3.0 IFrame code now uses default values if called with an old version of the host page script. Improved function naming. Old names have been deprecated and removed from docs, but remain in code for backwards compatabilty.
* v1.2.5 Fix publish to [plugins.jquery.com](https://plugins.jquery.com).
* v1.2.0 Added autoResize option, added height/width values to iFrame public size function, set HTML tag height to auto, improved documentation [All [Jure Mav](https://github.com/jmav)]. Plus setInterval now only runs in browsers that don't support [MutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver) and is on by default, sourceMaps added and close() method introduced to window.parentIFrame object in iFrame. 
* v1.1.1 Added event type to messageData object.
* v1.1.0 Added DOM [MutationObserver](https://developer.mozilla.org/en/docs/Web/API/MutationObserver) trigger to better detect content changes in iFrame, [#7](https://github.com/davidjbradshaw/iframe-resizer/issues/7) Set height of iFrame body element to auto to prevent resizing loop, if it's set to a percentage.
* v1.0.3 [#6](https://github.com/davidjbradshaw/iframe-resizer/issues/6) Force incoming messages to string. Migrated to Grunt 4.x. Published to Bower.
* v1.0.2 [#2](https://github.com/davidjbradshaw/iframe-resizer/issues/2) mime-type changed for IE8-10.
* v1.0.0 Initial published release.


## License
Copyright &copy; 2013-14 [David J. Bradshaw](https://github.com/davidjbradshaw).
Licensed under the [MIT license](http://opensource.org/licenses/MIT).
