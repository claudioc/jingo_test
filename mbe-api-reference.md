#MBE Api reference
Table of contents
---

* [Introduction](#introduction)
* [A tech prelude to the API](#tech-prelude)
* [Creating a MBE campaign using the generic embed and API](#by-generic-embed)
* [The API](#the-api)
  * [MBE.showVideo](#api-show-video)
  * [MBE.showPage](#api-show-page)
      - [The DUMMY adapter](#api-show-page-dummy)
      - [The HTML adapter](#api-show-page-html)
      - [The CPI adapter](#api-show-page-cpi)
      - [The BANNER adapter](#api-show-page-banner)
      - [How to customize a page adapter](#customize-adapter)
  * [&lt;adapter&gt;.on](#api-on)
  * [MBE.end](#api-end) (deprecated)
  * [MBE.finish](#api-finish)
  * [MBE.convert](#api-convert)
  * [MBE.payout](#api-payout)
  * [MBE.go](#api-go)
  * [MBE.track](#api-track)
* Advanced API
  * [MBE.isBrowser](#api-is-browser)
  * [MBE.loadJS](#api-load-js)
  * [MBE.getEnv](#api-get-env)
  * [MBE.setEnv](#api-set-env)
* [Complete examples](#complete-examples)

<a id="introduction"></a>
[Introduction](#introduction)
------------
The Mobile Brandengage JavaScript client (MBE) has two sets of "external" interfaces (API):
  1. one which is private and is used to control its internal behavior (think about it as the controller of the application)
  2. one which is used to program the behavior of the single "pages" presented to the end user and it is used inside the AMS when defining a landing page

The first set of API is not documented here and contains stuff like "request the offers", "start the offer" which are in use by the SDK (when the MBE is used along with an SDK, of course).

To give the maximum level of flexibility, the second set of API enables the offer manager to have complete control on what can happen during the engagement life cycle. This of course means that you have to "program" the behavior of the engagement itself: there are no defaults or implicit actions (except for the tracking): if you don't "program" your engagement, nothing will happen.

**UPDATE**: this is not entirely true anymore. There is at least one adapter (the CPI one) which automatically ends the engagement upon user action.

You define the engagement using AMS, more or less in the same way you define a _Steppy_ campaign. MBE has _steps_ (or _scenes_) which are defined in a template fashion: the campaign is composed by a number of scenes (usually 1 or 2, but there is no hard limit); you select the _type_ of the each scene and fill in the various fields needed for that type (es: the "url" for a YouTube kind of scene).

When the campaign will be presented, the system will _compile_ the template definition in the form of JavaScript using the MBE API, just like you'd do using a generic embed for the definition. 

<a id="by-generic-embed"></a>
## [Creating a MBE campaign using the generic embed and API](#by-generic-embed)
One of these types is the so called _Generic Embed_: with this type you will define every single aspect of the MBE scene, without any "help" by the templating system. You'll put everything you need inside it: HTML, CSS and of course JavaScript. You should then take care of each of those languages' specific syntax requirements. No validation is performed for the content of that textarea. The syntax and the logic of the content is your full responsibility.

The JavaScript you'll use inside the Generic Embed will use the MBE provided API.

**Important**: in theory you could create the _whole_ campaign using one generic embed only. You can - in fact - build up every single scene of the campaign directly from there (from one sigle generic embed field). This is how things worked _before_ the templating system.

<a id="tech-prelude"></a>
## [A tech prelude to the API](#tech-prelude)
for technical reasons, each method of the API needs to be prefixed by `MBE.` This is what we call "a namespace", and gives us the opportunity to live in a hostile environment. MBE, in turn, is just a shortcut to the full name of the namespace which is `Sponsorpay.MBE.API`. Use it if for some reason you cannot use "MBE".

The name of methods and the namespace(s) are case sensitive. Parameters in [square] brackets are optional.

<a id="the-api"></a>
## [Here comes the list of MBE API](#the-api)

<a id="api-show-video"></a>
### [`MBE.showVideo`](#api-show-video)

------------

Shows a video, given the video network and the URL and optionally starts it

*Usage*

`MBE.showVideo(adapter, url, [options]);`

The _name of the adapter_ can be one of (please use them lower cased):
- youtube
- html5
- unruly
- groupm
- dummy

The url must be any valid complete url for the video network (like http://www.youtube.com/watch?v=t-b-rxTtqvY for YouTube). If different kinds of url exist for the video network (as for YouTube) MBE will try its best to work around the differences. Just copy and paste the URL.

An _adapter_ is what contains the logic behind displaying a particular kind of video. The "unruly" and "groupm" adapters, indeed, automatically creates the HTML and JavaScript needed to show videos from the Unruly and GroupM video network.

The **dummy** adapter doesn't need any url or parameter (and does not display anything, of course, except for the 'This is a video' string). In one second it will fire the 'start', 'progress' and 'finish' event.

The options parameter must be specified as a JavaScript object with the following possible keys:
- autoplay (true or false, defaults to false) - many devices (like the iPhone) do not permit the usage of such parameter
- controls (true or false, defaults to false) - show or hide the controls (play, stop, pause)
- inline (true or false, defaults to false) - force the usage of the `webkit-playsinline` attribute in the HTML5 adapter

Note that additional options might be defined for each video network.

For the `groupm` adapter you'll need to pass three more parameters, which must be passed in the same way as the other one (like `autoplay` or `controls`):

- deviceId: if it's not used, a valid, fake deviceId will be generated
- placementId: if it's not used, a test value will be generated
- appleIdForAdvertisers: no default on this one

*Example*

```html
<script>
// Opens a video, no autoplay (default)
MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");

// Open a video in autoplay
MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY", {autoplay: true});

</script>
```

<a id="api-show-page"></a>
###[`MBE.showPage`](#api-show-page)

------------

The purpose of this method is to create and show a page. It uses a simple approach, similar to the `showVideo` method, where you give the method the name of the _adapter_ and the parameters needed for the adapter itself.

The showPage method returns a `page` object onto which you can listen for the `activate`, `close` and `abort` event.

*Usage*

`MBE.showPage(adapter name, [options]);`

*Page adapters*

The adapters at the moment defined are (the simplest first):

* dummy
* html
* cpi
* banner

<a id="api-show-page-dummy"></a>
####[`The Dummy adapter`](#api-show-page-dummy)

------------

The dummy adapter just displays an HTML `div` with a sample predefined text inside. No CSS styling is provided. The options parameter is not used.

*Example*

```html
<script>
var page = MBE.showPage("dummy");
page.on("activate", function() {
  console.log("Dummy page displayed");
})
</script>
```

<a id="api-show-page-html"></a>
####[`The HTML adapter`](#api-show-page-html)

------------

The purpose of this adapter is to display an HTML fragment provided by the author. This is the "raw" method of creating a page, where the author provides the full HTML page and the corresponding CSS. Via the options parameter, the author provides the `htmlId` of the fragment.
The fragment of html referred by the ID which defines the structure and the content of the page must already be present in the code.

You can define more that one page inside the code provided that each of them uses a different html ID.

The system will automatically add the `page` class to the element itself.

*Example*

```html
<style type="text/css">
#my-page {
  background-color: red;
}
</style>

<div id="my-page">
  <p>This is the content of the page</p>
</div>

<script>
var page = MBE.showPage("html", {htmlId: "my-page"});
page.on("activate", function() {
  console.log("Dummy page displayed");
})
</script>
```

<a id="api-show-page-cpi"></a>
####[`The CPI adapter`](#api-show-page-cpi)

------------

This adapter provides a way to render a fully fledged CPI page without having to write a single line of HTML or CSS.
Every possible _dynamic_ parameter is provided to the adapter via the `options` parameter, which is as usual a JavaScript object. Via the options parameter you define the _content_ and the _presentation_ part of the page, so this is the structure of the object itself.

As for the content, you need to provide:

* **title** which is the title of the page (or the title of the game)
* **publisher** the publisher name
* **price** the price, as a string. "Free" is also a valid value
* **icon** the complete URL of the icon for the software to be installed
* **installUrl** the complete URL of the install endpoint

As for the presentation, you need to provide:
* **bgPortrait** the complete URL of the background image for the portrait orientation
* **bgLandscape** the complete URL of the background image for the landscape orientation

If you don't have one of this data, just don't pass it to the function. It won't break anything.

*Example*

```html
<script>

var page = MBE.showPage("cpi", {

  content: {
    title: "Titolo del gioco",
    publisher: "Microsoft!",
    price: "Totally free",
    icon: "https://offerwall.s3.amazonaws.com/app_icons/1593/big_icon.png",
    // InstallUrl can be a string or an object, if you need to make things different for OSes
    installUrl: {
      android:  "http://zot.com/install.php?id=123",
      iOS: "something://somewhere"
    }
  },

  presentation: {
    bgPortrait: "http://offerwall.s3.amazonaws.com/assets/16622/Garfields_diner_hawaii_portrait_original.png",
    bgLandscape: "http://offerwall.s3.amazonaws.com/assets/16625/Garfields_diner_hawaii_landscape_original.png"
  },

  behavior: {
    preventDefault: false
  }
});

page.on("activate", function() {
  console.log("CPI page displayed");
})
</script>
```

The adapter will automatically create an HTML page with the following structure:

```html
<div.page.page-cpi>
  <div.content-wrapper>
    <div.content>
      <img.icon>
      <h1></h1>
      <span.publisher></span>
      <span.price></span>
      <a.action-button">
        <span></span>
      </a>
    </div>
  </div>
</div>
```

While the HTML of the page is hardcoded and not changeable, for the CSS part you have a lot of options. Please refer to the section [How to customize an adapter](#customize-adapter).

*Default behavior*

By default "clicking" on the "install button" will call the `MBE.end()` API method with the HREF of the button itself (which is - in fact - an A HTML element). Also, a pixel PAGE_CLICK is requested.
If you don't want this to happen (i.e., you want total control over what's happening), you can pass `preventDefault: true` behavior option to the API call. Note that the pixel will be requested anyway.


<a id="api-show-page-banner"></a>
####[`The BANNER adapter`](#api-show-page-banner)

------------

This adapter provides a way to render a fully fledged BANNER page (aka CPL) without having to write a single line of HTML or CSS. The page consists of a background image, a big button which moves the user to another web page, a title and a description.
Every possible _dynamic_ parameter is provided to the adapter via the `options` parameter, which is as usual a JavaScript object. Via the options parameter you define the _content_ and the _presentation_ part of the page, so this is the structure of the object itself.

As for the content, you need to provide:

* **title** the title of the page. It'll appear above the big button
* **description** the description of the page. It'll appear below the big button
* **buttonLabel** the label of the big button
* **url** where the user will be sent upon clicking on the button

As for the presentation, you need to provide:
* **bgPortrait** the complete URL of the background image for the portrait orientation
* **bgLandscape** the complete URL of the background image for the landscape orientation
* **buttonColor** the background color of the big button. You can specify a gradient specifying two colors, comma separated (i.e. "red, yellow" will create a gradient color with red on the top)
* **buttonLabelColor** the foreground color for the label in the button

If you don't have one of this data, just don't pass it to the function. It won't break anything.

*Example*

```html
<script>

var page = MBE.showPage("banner", {
  content: {
    title: "Microsoft North Korea",
    description: "One day we'll rule the world. For now we are stuck on the Moon. I'm not sure I like it, but... whatever.",
    buttonLabel: "Click here for the real deal",
    url: "http://google.com"
  },
  presentation: {
    themeHostname: "http://localhost:3000",
    buttonColor: "yellow,red",
    bgLandscape: "http://www.warrenphotographic.co.uk/photography/bigs/11634-Ginger-cat-white-background.jpg",
    buttonTextColor: "pink"
  });

page.on("activate", function() {
  console.log("BANNER page displayed");
});

page.on("userAction", function() {
  console.log("BANNER page has been clicked");
})
</script>
```

The adapter will automatically create an HTML page with the following structure:

```html
<div.page.page-banner>
  <div.content-wrapper>
    <div.content>
      <h1/>
      <a.action-button>
        <span/>
      </a>
      <p.description/>
    </div>
  </div>
</div>
```

While the HTML of the page is hardcoded and not changeable, for the CSS part you have a lot of options. Please refer to the section [How to customize an adapter](#customize-adapter).

*Default behavior*
When the users click on the big button the following actions will take place:

- the "userAction" event is fired (which can be trapped with the on() API method)
- the PAGE_CLICK pixel is requested
- the user is brought to the new page (after the pixel has been successfully fetched)

If you don't want to automatically follow the link (i.e., you want total control over what's happening), you can pass `preventDefault: true` behavior option to the API call (the userAction event will be fired anyway, and so will the PAGE_CLICK pixel).

Note that the engagement is **not** automatically ended (the user is not rewarded).

<a id="customize-adapter"></a>
###[`How to customize a page adapter`](#customize-adapter)

------------

**Customizing using themes**

When the system will show a page using an adapter (i.e.: cpi, banner, but not the html one) it will make use of a CSS file. This CSS file represents the current "theme" of the adapter. If, for example, you found yourself using two different graphic version of the same adapter, you could think about creating two different theme and use one or the other.

The system will automatically include the CSS file found in `/public/stylesheets/mbe/page_themes/cpi-default.css` of the OFW project. Here the `default` string refers to the name of the **theme** for the CPI page. If a different theme name is not specified, the system will load cpi-default.css.
You can use another file (think about it as "switching to another theme"), just using the `theme` property of the `presentation` property of the options object.

**Customizing using hand crafted CSS**

The portion of code which contains the adapter page definition is included inside the whole page as the last element. This means that you can add some custom CSS rules just inside it, and they will override the corresponding rules which are defined inside the theme file.

*Example*

```html
<style type="text/css">

  /* I want to use a google font */
  @font-face {
    font-family: 'Snowburst One';
    font-style: normal;
    font-weight: 400;
    src: local('Snowburst One'), local('SnowburstOne-Regular'), url(http://themes.googleusercontent.com/static/fonts/snowburstone/v1/zSQzKOPukXRux2oTqfYJjPn8qdNnd5eCmWXua5W-n7c.woff) format('woff');
  }

  .page {
    font-family: 'Snowburst One';
  }

  /* Don't want the content to have an opacity but a solid color */
  .page .content {
    opacity: 1;
    background-color: green;
  }
  .
</style>

<script>
  MBE.showPage("cpi", {
    content: {
      ... 
    },

    presentation: {
      // We also select the "funky" theme, which means we need to have a cpi-funky.css file in the theme location
      theme: "funky"
    }
  });
</script>
```

*Specifying a different location for the theme location*

While developing a new theme, it may happen that you need to "mix" different environments (like using the assets from your development machine but the database on staging). In these cases it might be useful to be able to specify a different location for the `theme location` for you pages.

You can do this using the `themeLocation` presentation option to specify the full path of the theme, or just `themeHostname` if you just need to change the hostname for the default theme location.

You also have the possibility to set the themeLocation and themeHostname "globally", so that any defined pages will use that information instead of having to set each one independently. To use this feature, just set the "pageThemeLocation" and "pageThemeHostname" parameters using the [`MBE.setEnv()`](#wiki-api-set-env) API.

```html

<script>

// Use this hostname for themes for every page
MBE.setEnv("pageThemeHostname", "http://localhost:3001");

MBE.showPage("cpi", {
  ...
});
</script>
```

*The default background*

The page will use the landscape background image as its "primary background" meaning that if this is not set, then the system will add the `default` class to the page which can be used to specify a default background image for this particular case (like `http://iframe.sponsorpay.com/images/mbe/def-bg.png`)

*Example*

```html
<script>
MBE.showPage("cpi", {

  // Let's specify an HTML id for the page (so we can target this one directly)
  htmlId: "no-logo-page",

  content: {
    ... 
  },

  presentation: {
    themeHostname: "localhost:3000"
    // ... or (don't mix themeHostname and themeLocation)
    themeLocation: "http://localhost:3000/themes"
  }

});
</script>
```

<a id="api-on"></a>
###[`<adapter>.on`](#api-on)

------------

The "on" method lets you listen to events that are fired on an adapter and runs some logic accordingly. The logic behind this feature can be summarized with "When something happens to the video or to the page, please execute this code". 

In this context the "object" is a video or a page. The method purposely resembles the same method present in jQuery for any other events (like "clicks"). The function that you define and which is called upon the event is fired is called the _event callback_.

It is important to understand that all your code should belong to an event callback. You want to execute some custom code of yours _only_ when you receive an input from the MBE client (namely, in the form of an event). Code which is put outside the event callbacks is run just after the whole textarea is injected into the page. You normally don't want this to happen.

*Usage*

`<adapter>.on(event name, function to run upon the event is fired);`

There is a predefined list of events that you can use. There is no validation on the event name, though: if you try to use an undefined event name, nothing will happen. This is the list of events:

- **activate**: on pages and videos, upon been shown
- **close**: on pages when the X button is used
- **finish**: on videos when a video is finished
- **abort**: on videos when a playing video is aborted using a X
- **start**: on videos when they start
- **error**: on videos when something bad happens
- **progress**: on videos upon a new quarter of the video has been shown (1, 2, 3, 4)
- **userAction**: on pages, once the user performed the required action (es: click the _install_ button)

*Example*

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");

video.on("start", function() {
  alert("Hurra! The video has been started");
});

video.on("error", function() {
  alert("Ops, something bad happened!");
});

video.on("progress", function(quarter) {
  MBE.track("PROGRESS_" + quarter);
});
</script>
```

You can mix events and other API call, of course:

```html
<div id="my-cpi-page" class="page">
  <p>This is the content of the page</p>
</div>

<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");

// Open the page as soon as the video is finished
video.on("finish", function() {
  var page = MBE.showPage("my-cpi-page");
});
</script>
```

You can receive parameters as well, in event callbacks:

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
video.on("progress", function(quarter) {
  alert("Video at its " + quarter + " quarter");
});
</script>
```

<a id="api-end"></a>
###[`MBE.end`](#api-end)

------------

**DEPRECATED** Ends and finalizes an engagement

*Usage*

`MBE.end([parameter]);`

**Note: this method is _deprecated_ and you should not use it anymore. Use `convert()`, `payout()` and `finish()` instead.**
You call this method when you want to consider the engagement finished. Ending the engagement means, among other things, that a _conversion_ has been made. 

Conceptually the `end` method does two things: (always) issues a conversion and (eventually) moves the user somewhere else. Depending on the value of the _parameter_, different behaviors may occur.

- if the parameter is a string, it will be considered an URL. That url will be followed rightly _after_ the server callback for the conversion has been executed. The url could be - for example - the install link for a CPI page.
- if the parameter has the `false` value, only the conversion will be made. No further actions will take place
- if the parameter is not used or it has the `true` value, the conversion will take place and either the CLOSE_FINISHED notification will be sent to the SDK or nothing happens (if we are inside a browser)

Note that checks are in place to make sure the conversion will happen only once (on the client side _and_ the server side).

*Examples*

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
// End the engagement as soon as the user watched the whole video
video.on("finish", function() {
  // Convert and move the user to anotherpage.com
  MBE.end("http://anotherpage.com");
});
</script>
```

```html
<div class="page">
  <a id="install-me" href="http://install/path">Click to install</a>
</div>

<script>
// End the engagement as soon as the user clicks on the install button
// This example uses jQuery, a library that you have always at your disposal
$('#install-me').on("click", function() {
  // Convert and move the user to the page pointed by the href of the button
  MBE.end(this.href);
  return false; // This is needed to instruct the browser to not follow the link
});
</script>
```

```html
<div class="page">
  <a id="install-me" href="http://install/path">Click to install</a>
</div>

<script>
// End the engagement as soon as the user clicks on the install button
// This example uses jQuery, a library that you have always at your disposal
$('#install-me').on("click", function() {
  // Convert and alert the SDK (or do nothing)
  MBE.end();
  return false; // This is needed to instruct the browser to not follow the link
});
</script>
```

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
// End the engagement as soon as the user watched the whole video
video.on("finish", function() {
  // Convert and nothing else if we are in browser
  if ( MBE.isBrowser() ) {
    MBE.end(false);
  }

  // Immediately show a page
  var page = MBE.showPage("my-page");
});
</script>
```

<a id="api-convert"></a>
###[`MBE.convert`](#api-convert)

------------

*Usage*

`MBE.convert([callback]);`

Will convert the engagement, meaning that a remote call to the "Advertiser and Payout callback" (`/cb`) will be issued. If the offer needs separate callbacks, only the Advertiser callback will be yielded (note that this business logic regarding the type of offer is implemented on the server, not the MBE client).

See also the `MBE.payout()` method.

An optional callback can be used to perform some operation after the server replied. Please note that the callback accept one parameter that can be used to understand if an error has been detected on the call.

*Example*

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
// End the engagement as soon as the user watched the whole video
video.on("finish", function() {

  // This will work, plain and simple
  MBE.convert();
  MBE.finish();

  // You can have a better control, if you want:
  MBE.convert(function(error) {
    
    if (!error) {
      MBE.finish();
    } else {
      alert("Sorry, an error occurred");
    }

  });
});
</script>
```

<a id="api-payout"></a>
###[`MBE.payout`](#api-payout)

------------

*Usage*

`MBE.payout([callback]);`

Will call the remote "Publisher Payout" callback (`/pp`)

See also the `MBE.convert()` method.

An optional callback can be used to perform some operation after the server replied. Please note that the callback accept one parameter that can be used to understand if an error has been detected on the call.

*Example*

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
// End the engagement as soon as the user watched the whole video
video.on("finish", function() {

  // This will work, plain and simple
  MBE.payout();
  MBE.finish();
});
</script>

<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
video.on("finish", function() {
  // You can have a better control, if you want:
  MBE.payout(function(error) {
    
    if (!error) {
      MBE.finish();
    } else {
      alert("Sorry, an error occurred");
    }

  });
});
</script>
```

<a id="api-finish"></a>
###[`MBE.finish`](#api-finish)

------------

*Usage*

`MBE.finish([url]);`

Just "moves" the user away, optionally following the url. If this method is not passed a URL, the method will notify the SDK with a `CLOSE_FINISHED` message. If the engagement is running inside a browser (no SDK) and the url is not used, nothing will happen.

*Example*

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
video.on("finish", function() {
  // Once the video is finished, move the user to Google
  MBE.finish("http://www.google.com");
});
</script>
```

```html
<script>
var video = MBE.showVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
video.on("finish", function() {
  // Once the video is finished, reward the user and do the default operation
  MBE.convert();
  MBE.finish();
});
</script>
```

<a id="api-go"></a>
###[`MBE.go`](#api-go)

------------

When working with independent scenes (i.e.: not using a single generic embed for the whole campaign), you can use the `go` API to go either to the next scene (just call it without parameter) or to a specific one (pass the 1-based index to the API call).

*Usage*

`MBE.go([index]);`

*Example*

```javascript
<script>
// This "generic embed" scene is the first of a two scenes engagement
var video = MBE.startVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");
video.on("finish", function() {
  // After the video finishes, just go the next scene
  MBE.go();
});
<script>
```

<a id="api-track"></a>
###[`MBE.track`](#api-track)

------------

One of the things that the MBE client addresses by itself is tracking (which means fetching the _mbe.gif_ or the _ofw.gif_ pixels when something noteworthy is happening). If you need to track any other situation you can use this API method for that. When present, the string "&#91;&#91;step&#93;&#93;" inside the action name (like "IMPRESSION_&#91;&#91;step&#93;&#93;_PAGE) is replaced with current step the MBE stage is showing at the moment.

The pixel name used for MBE is (quite understandably) `mbe.gif`.

Please refer to the [official documentation](
https://docs.google.com/a/sponsorpay.com/spreadsheet/ccc?key=0Am8fsimfDJgMdHZjLURlakFuX1Y1UWNhdG5uMjFpREE#gid=0) for the pixels automatically fetched (search for the `mbe.gif` pixel).

A graphical overview of the current state of tracking can be found [here](https://www.dropbox.com/s/k283fy6yvxr7zit/diagram.png) (warning: might be obsolete).

*Usage*

`MBE.track(action, [url]);`

The action can be any string, except the specially meaningful value of `custom`. The url is needed only if the action is "custom". In that case, the url is fetched and no other pixels are fetched (this is useful when we need to fetch a third party pixel).

This method will fetch the `mbe.gif` url, **not** the `ofw.gif` one.

[[Important]] Please note that every action will be also automatically prefixed by the `OFFER_ENGAGEMENT_[step]` string.

Be sure to have also read the [Preliminary note about tracking](#tracking-note).

*Example*

```html
<script>
var video = MBE.startVideo("youtube", "http://www.youtube.com/watch?v=t-b-rxTtqvY");

// Plot a pixel when the video starts
video.on("start", function() {
  MBE.track("THERMONUCLER_GLOBAL_WAR_HAS_STARTED");
});

// Plot a custom pixel when the video ends
video.on("finish", function() {
  MBE.track("custom", "http://thermonuclearwarscounter.com/track.gif?key=324");
});
</script>
```

<a id="advanced-api"></a>
## [Advanced API](#advanced-api)

There is also a set of _advanced_ API that might be useful in some occasion.

<a id="api-is-browser"></a>
### [MBE.isBrowser](#api-is-browser)

------------

Returns true if the engagement is running inside a browser (Safari). Please note that MBE is supposed to work only inside WebKit powered browsers.

<a id="api-get-env"></a>
### [MBE.getEnv](#api-get-env)

------------

Retrieves the value of an internal variable

*Usage*

`MBE.getEnv(name);`

We are not disclosing at the moment the full list of possible variables names but you might be interested in:
- `integration` (integration name)
- `isBrowser` (are we running inside a browser or inside a webview?)

<a id="api-set-env"></a>
### [MBE.setEnv](#api-set-env)

------------

Set the value of an internal variable

*Usage*

`MBE.setEnv(name, value);`

<a id="complete-examples"></a>
##[Complete examples](#complete-examples)

Show an Unruly video and issue a conversion at the end (note that there is no need for HTML or CSS, just the control script):

```html

<script>

  var video = MBE.showVideo("unruly", "http://video.unrulymedia.com/wildfire_85460297.js");

  video.on("finish", function() {
    // Using a callback ensures that we also get a response from the server, before finish()-ing
    MBE.convert(function() {
      MBE.finish();
    });
  });
  
</script>

```

Show an YouTube video in autoplay and show a CPI page at the end

```html
<style type="text/css">
.page {
  background:url(http://coolbuddy.com/imgs/dragon.jpg) no-repeat;
  background-size: contain;
}
</style>

<div class="page" id="cpi_offer">
  <p>This is the content</p>
  <a href="http://path/to/install">Click here to install</a>
</div>

<script>
  var video = MBE.showVideo("youtube", {url: "http://www.youtube.com/watch?v=tgiosOKJu28", autoplay: true});

  video.on("finish", function() {
    MBE.payout();
    var page = MBE.showPage("cpi_offer");
  });
  
  $("#cpi_offer a").on("click", function() {
    var url = this.href;
    // Convert and move the user to url pointed by the button
    MBE.convert(function() {
      MBE.finish(url);
    });
    return false; // This is needed to instruct the browser to not follow the link
  });
</script>

```

<a id="api-load-js"></a>
###[`MBE.loadJS(script_url, fn)`](#api-load-js)
Sometimes, when integrating a complex third party script inside a scene, it would be necessary to include another JavaScript file inside it. In this case, DO NOT USE the `<script>` tag (it won't work anyway), but instead use the provided `loadJS` API method.
After the script has been successfully loaded, the `fn` callback will be called.

Example:
```javascript
// This integration requires swfobject
MBE.loadJS("//ajax.googleapis.com/ajax/libs/swfobject/2.2/swfobject.js", function() {
  // The rest of the integration code which uses the library
  ...
});
```