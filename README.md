# What Is The Asset Pipeline

## Objectives

1. Understand the 4 main features of the asset pipeline.
1. Identify the Asset Paths
2. Know how Asset Manifests provide concatenation of CSS and JS.
3. Use preprocessing languages like SASS or CoffeeScript
5. Define Asset Fingerprinting

## Outline

For a long time we treated Javascript and CSS as an after thought in developing web applications. All of our asset code, things like images, stylesheets, and javascript were kept in a massive folder called public and served outside of the context of our Rails application. As the web evolved, that no longer made sense.

The asset pipeline is the Rails answer to managing stylesheets, javascripts, and images.

## Asset Paths

The first problem it address is where to put things. We have to keep things very organized in our application but by keeping separate files and folders for each concept or unit of code, we have 2 problems.

1. How do we know where things are? Is the calendar library in app/assets/javascripts/calendar.js or vendor/javascripts/calendar.js?

2. We don't want to serve each file separately as that is very slow to the browser. It makes sense for us to maintain separate small files for readability and organization but for the browser, we'd rather smash all those small files together and let the browser only load 1 JS file and 1 CS file. That is called concatenation.

The Asset Pipeline gives our application a concept of Asset Paths. Just like in BASH we have a PATH that is a combination of folders to look for files in, the Asset Path is a combination of folders to look for assets in.

Rails.application.assets.paths
=> ["/Users/avi/asset-test/app/assets/images", "/Users/avi/asset-test/app/assets/javascripts", "/Users/avi/asset-test/app/assets/stylesheets", "/Users/avi/asset-test/vendor/assets/javascripts", "/Users/avi/asset-test/vendor/assets/stylesheets", "/Users/avi/.rvm/gems/ruby-2.2.3/gems/turbolinks-2.5.3/lib/assets/javascripts", "/Users/avi/.rvm/gems/ruby-2.2.3/gems/jquery-rails-4.1.0/vendor/assets/javascripts", "/Users/avi/.rvm/gems/ruby-2.2.3/gems/coffee-rails-4.1.1/lib/assets/javascripts"]

If we put any asset in any of those folders, we can access them via the URL '/assets' in our application

Quick test of a Rails application with some JS in app/assets/javascripts and vendor/assets/javascript and loading both files via /assets/file_in_app_assets.js vs /assets/file_in_vendor_assets.js - it doesn't matter where the file is, any request to /assets will trigger our asset lookup in our asset paths.

You can also add folders to your asset paths by Rails.application.assets.paths << Path to new folder

So that's the first feature, we can put assets anywhere and access them via a single /assets URL.

## Manifests and Concatenation

Now that we can put files anywhere, how do we get them all together again?

The asset pipeline has a notion of a manifest file for stylesheets and javascript.

A manifest is one file with magic CSS or Javascript files that turn the file into a place from where to require and load other files. This isn't a feature of JS or CSS but rather the asset pipeline. You can have a manifest that looks like:

File: app/assets/javascripts/application.js
```
//= require jquery
//= require calendar
```

When you include that single file in your layout with javascript_include_tag, the asset pipeline will look for all of those files anywhere in the asset path (so notice that we say require calendar - which lives in app/assets/javascripts/calendar) - you can require anything from the same level of the asset paths - if you can access it via /assets/calendar.js you can require it via 'calendar'

These manifests will concatenate the files into one file in production but mutliple in development for debugging.

The sprocket directives that power our asset manifests will be covered in detail later.

## Preprocessing

Because we're loading assets now through rails, we can preprocess the files using popular languages like SASS for writing better CSS. If you make an asset, like theme.css.sass you are telling the asset pipeline that before serving this theme.css, it must pass through the SASS processor to be compiled into CSS. We get this for free.

## Fingerprinting

Fingerprinting is a technique that makes the name of a file dependent on the contents of the file. When the file contents change, the filename is also changed. For content that is static or infrequently changed, this provides an easy way to tell whether two versions of a file are identical, even across different servers or deployment dates.

When a filename is unique and based on its content, HTTP headers can be set to encourage caches everywhere (whether at CDNs, at ISPs, in networking equipment, or in web browsers) to keep their own copy of the content. When the content is updated, the fingerprint will change. This will cause the remote clients to request a new copy of the content. This is generally known as cache busting.

The technique sprockets uses for fingerprinting is to insert a hash of the content into the name, usually at the end. For example a CSS file global.css

global-908e25f4bf641868d8683022a5b62f54.css
This is the strategy adopted by the Rails asset pipeline.

Rails' old strategy was to append a date-based query string to every asset linked with a built-in helper. In the source the generated code looked like this:

/stylesheets/global.css?1309495796
The query string strategy has several disadvantages:

Not all caches will reliably cache content where the filename only differs by query parameters

Steve Souders recommends, "...avoiding a querystring for cacheable resources". He found that in this case 5-20% of requests will not be cached. Query strings in particular do not work at all with some CDNs for cache invalidation.

The file name can change between nodes in multi-server environments.

The default query string in Rails 2.x is based on the modification time of the files. When assets are deployed to a cluster, there is no guarantee that the timestamps will be the same, resulting in different values being used depending on which server handles the request.

Too much cache invalidation

When static assets are deployed with each new release of code, the mtime (time of last modification) of all these files changes, forcing all remote clients to fetch them again, even when the content of those assets has not changed.

Fingerprinting fixes these problems by avoiding query strings, and by ensuring that filenames are consistent based on their content.

Fingerprinting is enabled by default for production and disabled for all other environments. You can enable or disable it in your configuration through the config.assets.digest option.

## Conclusion

The asset pipeline is a lot at once and it's hard, especially debugging. Try to focus on the problems it is solving.

1. Asset Paths
2. Manifests and Concatenation
3. Preprocessing
4. Fingerprinting

Finally, watch tThe DHH Keynote where he introduces the asset pipeline, it's good, promise. https://www.youtube.com/watch?v=cGdCI2HhfAU
