---
layout: post
title: "localForage: Offline Storage, Improved"
description: "Learn how to store data offline in a web browser without breaking your brain."
modified: 2014-10-05
tags: [code, development, offline]
location: Montreal, Canada
image: /assets/article_images/2014-02-12-localforage-offline-storage-improved/cover.jpg
credit: Timo Heuer
creditlink: http://www.flickr.com/photos/upim/3569619036/
---

Web apps have had offline capabilities like saving large data sets and binary files for some time. You can even do things like [cache MP3 files](https://github.com/mozilla/high-fidelity). Browser technology can store data offline and plenty of it. The problem, though, is that the technology choices for how you do this are fragmented.

localStorage gets you really basic data storage, but it's slow and can't handle binary blobs. IndexedDB and WebSQL are asynchronous, fast, and support large data sets, but their APIs aren't very straightforward. Even still, neither IndexedDB nor WebSQL have support from all of the major browser vendors and that doesn't seem like something that will change in the near future.

If you need to write a web app with offline support and don't know where to start, then this is the article for you. If you've ever tried to start working with offline support but it made your head spin, this article is for you too. Mozilla has made a library called [localForage](https://github.com/mozilla/localForage) that makes storing data offline in *any* browser a much easier task.

[around](https://github.com/mozilla/around) is an HTML5 Foursquare client I wrote that helped me work through some of the pain points of offline storage. We're still going to walk through how to use localForage, but there's some source for those of you that like learn by perusing code.

localForage is a JavaScript library that uses the very simple [localStorage API](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Storage#localStorage). localStorage gives you, essentially, the features of get, set, remove, clear, and length, but adds:

* an asynchronous API with callbacks
* IndexedDB, WebSQL, and localStorage drivers (managed automatically; the best driver is loaded for you)
* Blob and arbitrary type support, so you can store images, files, etc.
* support for ES6 Promises

The inclusion of IndexedDB and WebSQL support allows you to store more data for your web app than localStorage alone would allow. The non-blocking nature of their APIs makes your app faster by not hanging the main thread on get/set calls. Support for [promises](http://promises-aplus.github.io/promises-spec/) makes it a pleasure to write JavaScript without callback soup. Of course, if you're a fan of callbacks, localForage supports those too.

## Enough talk; show me how it works!

The traditional localStorage API, in many regards, is actually very nice; it's simple to use, doesn't enforce complex data structures, and requires *zero* boilerplate. If you had a configuration information in an app you wanted to save, all you need to write is:

{% highlight js %}
// Our config values we want to store offline.
var config = {
    fullName: document.getElementById('name').getAttribute('value'),
    userId: document.getElementById('id').getAttribute('value')
};

// Let's save it for the next time we load the app.
localStorage.setItem('config', JSON.stringify(config));

// The next time we load the app, we can do:
var config = JSON.parse(localStorage.getItem('config'));
{% endhighlight %}

Note that we need to save values in localStorage as strings, so we convert to/from JSON when interacting with it.

This appears delightfully straightforward, but you'll immediately notice a few issues with localStorage:

1. **It's synchronous.** We wait until the data has been read from the disk and parsed, regardless of how large it might be. This slows down our app's responsiveness. This is especially bad on mobile devices; the main thread is halted until the data is fetched, making your app seem slow and even unresponsive.

2. **It only supports strings.** Notice how we had to use `JSON.parse` and `JSON.stringify`? That's because localStorage only supports values that are JavaScript strings. No numbers, booleans, Blobs, etc. This makes storing numbers or arrays annoying, but effectively makes storing Blobs impossible (or at least VERY annoying and slow).

## A better way with localForage

localForage gets past both these problems by using asynchronous APIs but with localStorage's API. Compare using IndexedDB to localForage for the same bit of data:

### IndexedDB Code

{% highlight js %}
// IndexedDB.
var db;
var dbName = "dataspace";

var users = [ {id: 1, fullName: 'Matt'}, {id: 2, fullName: 'Bob'} ];

var request = indexedDB.open(dbName, 2);

request.onerror = function(event) {
    // Handle errors.
};

request.onupgradeneeded = function(event) {
    db = event.target.result;

    var objectStore = db.createObjectStore("users", { keyPath: "id" });

    objectStore.createIndex("fullName", "fullName", { unique: false });

    objectStore.transaction.oncomplete = function(event) {
        var userObjectStore = db.transaction("users", "readwrite")
                                .objectStore("users");
    }
};

// Once the database is created, let's add our user to it...
var transaction = db.transaction(["users"], "readwrite");

// Do something when all the data is added to the database.
transaction.oncomplete = function(event) {
    console.log("All done!");
};

transaction.onerror = function(event) {
    // Don't forget to handle errors!
};

var objectStore = transaction.objectStore("users");

for (var i in users) {
    var request = objectStore.add(users[i]);
    request.onsuccess = function(event) {
        // Contains our user info.
        console.log(event.target.result);
    };
}
{% endhighlight %}

WebSQL wouldn't be quite as verbose, but it would still require a fair bit of boilerplate. With localForage, you get to write this:

### localForage Code

{% highlight js %}
// Save our users.
var users = [ {id: 1, fullName: 'Matt'}, {id: 2, fullName: 'Bob'} ];
localforage.setItem('users', users, function(err, result) {
    console.log(result);
});
{% endhighlight %}

That was a bit less work.

## Data other than strings

Let's say you want to download a user's profile picture for your app and cache it for offline use. It's easy to save binary data with localForage:

{% highlight js %}
// We'll download the user's photo with AJAX.
var request = new XMLHttpRequest();

// Let's get the first user's photo.
request.open('GET', "/users/1/profile_picture.jpg", true);
request.responseType = 'arraybuffer';

// When the AJAX state changes, save the photo locally.
request.addEventListener('readystatechange', function() {
    if (request.readyState === 4) { // readyState DONE
        // We store the binary data as-is; this wouldn't work with localStorage.
        localforage.setItem('user_1_photo', request.response, function() {
            // Photo has been saved, do whatever happens next!
        });
    }
});

request.send()
{% endhighlight %}

Next time we can get the photo out of localForage with just three lines of code:

{% highlight js %}
localforage.getItem('user_1_photo', function(err, photo) {
    // Create a data URI or something to put the photo in an <img>
    // tag or similar.
    console.log(photo);
});
{% endhighlight %}

## Callbacks and promises

If you don't like using callbacks in your code, you can use [ES6 Promises](http://www.promisejs.org/) instead of the callback argument in localForage. Let's get that photo from the last example, but use promises instead of a callback:

{% highlight js %}
localforage.getItem('user_1_photo').then(function(photo) {
    // Create a data URI or something to put the photo in an <img>
    // tag or similar.
    console.log(photo);
});
{% endhighlight %}

Admittedly, that's a bit of a contrived example, but [around](https://github.com/mozilla/around) has some [real-world code](https://github.com/mozilla/around/blob/7b23feca9dcd35d71ceb85de3d9492af2fa490f6/www/js/app.coffee#L18) if you're interested in seeing the library in everyday usage.

## Cross-browser support

**localForage supports all modern browsers**. IndexedDB is [available in all modern browsers aside from Safari](http://caniuse.com/indexeddb) (IE 10+, IE Mobile 10+, Firefox 10+, Firefox for Android 25+, Chrome 23+, Chrome for Android 32+, and Opera 15+). Meanwhile, the stock Android Browser (2.1+) and Safari use WebSQL.

In the worst case, localForage will fall back to localStorage, so you can at least store basic data offline (though not blobs and much slower). It at least takes care of automatically converting your data to/from JSON strings, which is how localStorage needs data to be stored.

[Learn more about localForage on GitHub](https://github.com/mozilla/localForage), and please file issues if you'd like to see the library do more!

----

<small>This article was edited by [Angelina Fabbro](http://realityhacking.net/) and [Robert Nyman](http://robertnyman.com/), with notes from [Fred Wenzel](http://fredericiana.com/).<br> It was originally published on the [Mozilla Hacks Blog](https://hacks.mozilla.org/2014/02/localforage-offline-storage-improved/).</small>
