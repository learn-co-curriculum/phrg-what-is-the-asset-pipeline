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

A lot of files go into creating web applications. The CSS and JavaScript files alone can be hard to organize. What folders do we create? What files go where? The Asset Pipeline provides an answer for this problem. We have to keep things very organized in our application but by keeping separate files and folders for each concept or unit of code, we have 2 problems.

1. How Rails know where things are? Is the calendar library in app/assets/javascripts/calendar.js or vendor/javascripts/calendar.js?

2. We don't want to serve each file separately as this will make our page load very slow in the browser. It makes sense for us to maintain separate small files for readability and organization but for the browser, we'd rather smash all those small files together and load 1 JS file and 1 CS file. This process is called concatenation.

Let's talk about our first problem, who does Rails know where to look? The Asset Pipeline gives our application a concept of Asset Paths. Just like in BASH we have a PATH environment variable that is a combination of folders to look for files in, the Asset Path is a combination of folders to look for assets in. Let's take a look at an example of how our Asset Path can be configured.

```ruby
Rails.application.config.assets.paths =>
["/Users/avi/asset-test/app/assets/images",
"/Users/avi/asset-test/app/assets/javascripts",
"/Users/avi/asset-test/app/assets/stylesheets",
"/Users/avi/asset-test/vendor/assets/javascripts",
"/Users/avi/asset-test/vendor/assets/stylesheets",
"/Users/avi/.rvm/gems/ruby-2.2.3/gems/turbolinks-2.5.3/lib/assets/javascripts",
"/Users/avi/.rvm/gems/ruby-2.2.3/gems/jquery-rails-4.1.0/vendor/assets/javascripts",
"/Users/avi/.rvm/gems/ruby-2.2.3/gems/coffee-rails-4.1.1/lib/assets/javascripts"]
```
If we put an asset in any of these folders, we can access them via the URL '/assets' in our application. 

**Quick test of a Rails application with some JS in app/assets/javascripts and vendor/assets/javascript and loading both files via /assets/file_in_app_assets.js vs /assets/file_in_vendor_assets.js - it doesn't matter where the file is, any request to /assets will trigger our asset lookup in our asset paths.**

If you have addition folders for Rails to search, you can add the folders to your asset paths. This is normally done in the file`config/initializers/assets.rb`.

```ruby
Rails.application.config.assets.paths << "New Path"
```

That's pretty configurable. We can put assets anywhere, configure our Asset Path and access them via a single /assets URL.

## Manifests and Concatenation

Now that we can put files anywhere, how do we get them to be included on our web pages? The Asset Pipeline uses a manifest file to tell Rails what to load.

This manifest file is a centeral location where we can list all the CSS and JS files our application needs. This isn't a feature of JS or CSS but rather the asset pipeline. Here is an example of what our manifest file looks like:

File: app/assets/javascripts/application.js
```
//= require jquery
//= require calendar
```
When you include the manifest file in your layout with javascript_include_tag, the asset pipeline will look for all of the files listed here in the asset path. Notice how we require calendar. This file lives in app/assets/javascripts/calendar yet we only specified the name and not the full path. The Asset Pipeline will search all the paths it has for a file with the name we provided.

--- HERE NOW --- 
Now that we solved the question of deiscoverablility let's talk about concatentation. Like we discussed earlier, we don't to load our files in the browser one by one. It's better to perform one download then a bunch of small downloads from our browser. The manifests files we configure in Rails will automatically concatenate the files listed in them into one file in production. When we are developing our application this might not be the best option since it can make debugging hard but Rails will actually server each file separately when we are running in development mode. No need to do anything.

Finally, the sprocket directives that power our asset manifests will be covered in detail later.

## Preprocessing

Being able to combine files and load them from a set of pre defined locations in our application is a great beneifit of the Asset Pipeline. That's only the beginning. Because we're loading assets through rails, we can preprocess the files using popular languages like SCSS for writing better CSS. If you make an asset named theme.css.scss, you are telling the asset pipeline that before serving theme.css to the browser, it must pass through the SCSS processor which will compiled it into CSS. The only thing we had to do was provide the correct file extension, `.scss` to the file.

## Fingerprinting

The last benifit we will talk about is Fingerprinting but first let's talk about the problem is helps us solve. When we server files to the browser, they are likely to be cached to avoid downloading them again in the future. This saves bandwidth for us and provides a speed boost for the user. This is great until you change the file and you want all of your users to get the new one instead of the cached one they have stored locally. If the updated file is named the same as the old file the browser won't request the new one. We need a way to change the file name under the hood with out us having to do it.

Fingerprinting is a technique that makes the name of a file dependent on the contents of the file. When the file contents change, the filename is also changed. For content that is static or infrequently changed, this provides an easy way to tell whether two versions of a file are identical, even across different servers or deployment dates.

When a filename is unique and based on its content, HTTP headers can be set to encourage caches everywhere (whether at CDNs, at ISPs, in networking equipment, or in web browsers) to keep their own copy of the content. When the content is updated, the fingerprint will change. This will cause the remote clients to request a new copy of the content. This is known as cache busting.

The technique sprockets uses for fingerprinting is to insert a hash of the content into the end of the file name. For example, take this CSS file name global.css. Sprockets will add the hash `908e25f4bf641868d8683022a5b62f54` to the end of the file name like this:
```
global-908e25f4bf641868d8683022a5b62f54.css
```

If you happened to be using an older version of Rails (Rails 2.x), the strategy used was to append a date-based query string to every asset linked with a built-in helper. This looked like this:
```
global.css?1309495796
```

The query string strategy has several disadvantages:

- Not all caches will reliably cache content where the filename only differs by query parameters

Steve Souders recommends, "...avoiding a querystring for cacheable resources".  5-20% of your requests will not be cached. Query strings in particular do not work at all with some CDNs for cache invalidation.

- The file name can change between nodes in multi-server environments.

The default query string in Rails 2.x is based on the modification time of the files. When assets are deployed to a cluster, there is no guarantee that the timestamps will be the same, resulting in different values being used depending on which server handles the request.

- Too much cache invalidation

When static assets are deployed with each new release of code, the mtime (time of last modification) of all these files changes, forcing all remote clients to fetch them again, even when the content of those assets has not changed.

Fingerprinting fixes all these problems by ensuring that filenames are consistent based on their content.

Fingerprinting is enabled by default for production and disabled for all other environments. You can enable or disable it in your configuration through the `config.assets.digest` option.

## Conclusion

The Asset Pipeline is definitely more complex then just serving assets from a public folder and it can be hard to debug. Learning how to use it will pay off in the long run by saving us time and headaches. Just think about all the problems it solves for us.

1. Asset Paths
2. Manifests and Concatenation
3. Preprocessing
4. Fingerprinting

Finally, definitely check out the DHH Keynote where he introduces the asset pipeline, https://www.youtube.com/watch?v=cGdCI2HhfAU. It's good, I promise.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/what-is-the-asset-pipeline' title='What Is The Asset Pipeline'>What Is The Asset Pipeline</a> on Learn.co and start learning to code for free.</p>
