# Gulp + Browserify: The Everything Post

## Getting Started

One weekend, I decided to really immerse myself in Grunt and RequireJS. Gotta
stay up on these things right? Done. Then Monday rolls around,
[“and just like that Grunt and RequireJS are out, it’s all about Gulp and Browserify now.”][1]

**(╯°□°）╯︵ ┻━┻**

When I was done flipping tables, I set aside my newly acquired Grunt +
RequireJS skills, and started over again with Gulp and Browserify to see what 
all the fuss was about.

**You guys. The internet was right**. To save you some googling, doc crawling,
and trial and error I went through, I've assembled some resources and 
information I think you'll find helpful in getting started.

 **┬─┬ノ( º _ ºノ)** 

### [Gulp + Browserify starter repo][2]

I've created a [Gulp + Browserify starter repo][2] with examples of how to
accomplish some common tasks and workflows.

### [Frequently Asked Questions Wiki][3]

[Node][4], [npm][5], [CommonJS Modules][6], [package.json][7]…wat? When I
dove into this stuff, much of the documentation out there assumed a familiarity 
with things with which I was not at familiar all. I've compiled some background 
knowledge into a[FAQ Wiki][3] attached to the above mentioned starter repo to
help fill in any knowledge gaps.

## Why Gulp is Great

### It makes sense.

I picked up Gulp just days after learning Grunt. For whatever reason, I found
Gulp to be immediately easier and more enjoyable to work with. The idea of 
piping a stream a files through different processes makes a lot of sense.

> gulp's use of streams and code-over-configuration makes for a simpler and
> more intuitive build.
>[- gulpjs.com][8]

Here's what a basic image processing task might look like:

    var gulp       = require('gulp');
    var imagemin   = require('gulp-imagemin');
    
    gulp.task('images', function(){
        return gulp.src('./src/images/**')
            .pipe(imagemin())
            .pipe(gulp.dest('./build/images'));
    });
    

First, `gulp.src` sucks in a stream of files and gets them ready to be piped
through whatever tasks you've made available. In this instance, I'm running all 
the files through`gulp-imagemin`, then outputting them to my `build` folder
using`gulp.dest`. To add additional processing (renaming, resizing,
liveReloading, etc.), just tack on more pipes with tasks to run.

### Speed!

It's really fast! I just finished building a fairly complex JS app. It handled
compiling SASS, CoffeeScript with source maps, Handlebars Templates, and running
LiveReload like it was no big deal.

> By harnessing the power of node's streams you get fast builds that don't
> write intermediary files to disk.
>[- gulpjs.com][8]

### This killer way to break up your gulpfile.js

A `gulpfile` is what gulp uses to kick things off when you run `gulp`. If you'
re coming from Grunt, it's just like a`gruntfile`. After some experimenting,
some Pull Request suggestions, and learning how awesome Node/CommonJS modules 
are (more on that later), I broke out all my tasks into individual files, and 
came up with this`gulpfile.js`. I'm kind of in love with it.

    var gulp = require('./gulp')([
        'browserify',
        'compass',
        'images',
        'open',
        'watch',
        'serve'
    ]);
    
    gulp.task('build', ['browserify', 'compass', 'images']);
    gulp.task('default', ['build', 'watch', 'serve', 'open']);
    

~200 characters. So clean, right? Here's what's happening: I'm requireing a
gulp module I've created at`./gulp/index.js`, and am passing it a list of tasks
that correspond to task files I've saved in`./gulp/tasks`.

    var gulp = require('gulp');
    
    module.exports = function(tasks) {
        tasks.forEach(function(name) {
            gulp.task(name, require('./tasks/' + name));
        });
    
        return gulp;
    };
    

For each task name in the array we're passing to this method, a `gulp.task`
gets created with that name, and with the method exported by a file of the same 
name in my`./tasks/` folder. Now that each individual task has been registered
, we can use them in the bulk tasks like`default` that we defined at the bottom
of our`gulpfile`.

![folder structure][9]

This makes reusing and setting up tasks on new projects really easy. 
[Check out the starter repo][2], and [read the Gulp docs][10] to learn more.

## Why Browserify is Great

> “Browserify lets you require('modules') in the browser by bundling up all
> of your dependencies.
> ” - *[Browserify.org][11]*

Browserify looks at a single JavaScript file, and follows the `require`
dependency tree, and bundles them into a new file. You can use Browserify on the
command line, or through its API in Node (using Gulp in this case
).

#### Basic API example

**app.js**

    var hideElement = require('./hideElement');
    
    hideElement('#some-id');
    

**hideElement.js**

    var $ = require('jquery');
    module.exports = function(selector) {
        return $(selector).hide();
    };
    

**gulpfile.js**

    var browserify = require('browserify');
    var bundle = browserify('./app.js').bundle()
    

Running `app.js` through Browserify does the following:

1.  See that `app.js` requires `hideElement.js` 
2.  See that `hideElement.js` requires a module called `jquery` 
3.  Bundles together jQuery, hideElement.js, and app.js into one file, making
    sure each dependency is available when and where it needs to be.
   

### CommonJS > AMD

Our team had already moved towards module-based js with [Require.js][12] and 
[Almond.js][13], which both are implementations of the [AMD module pattern][14]

**AMD / RequireJS Modules felt cumbersome and awkward.**

    require([
        './thing1',
        './thing2',
        './thing3'
    ], function(thing1, thing2, thing3) {
        // Tell the module what to return/export
        return function() {
            console.log(thing1, thing2, thing3);
        };
    });
    

**The first time using CommonJS (Node) modules was a breath of fresh air.**

    var thing1 = require('./thing1');
    var thing2 = require('./thing2');
    var thing3 = require('./thing3');
    
    // Tell the module what to return/export
    module.exports = function() {
        console.log(thing1, thing2, thing3);
    };
    

Make sure to read up on 
[how `require` calls resolve to files, folders, and node_modules.][15]

### Browserify is awesome because Node and NPM are awesome.

Node uses the CommonJS pattern for requiring modules. What really makes it
powerful though is the ability to quickly install, update, and manage 
dependencies with[Node Package Manager][16] (npm). Once you've tasted this
combination, you'll want that power for always. Browserify is what lets us have 
it in the browser.

[Say you need jQuery.][17] Traditionally, you might open you your browser, find
the latest version on jQuery.com, download the file, save it to a vendor folder,
then add a script tag to your layout, and let it attach itself to`window` as a
global object.

With npm and Browserify, all you have to do is this:

*Command Line*

    npm install jquery --save
    

*app.js*

    var $ = require('jquery');
    $('.haters').fadeOut();
    

This fetches the latest version of jQuery from NPM, and downloads the module
into a node_modules folder at the root of your project. The`--save` flag
automatically adds the package to your`dependencies` object in your 
`package.json` file. Now you can `require('jquery')` in any file that needs it
. The`jQuery` object gets exported locally to `var $`, instead of globally on
`window`. This was especially nice when I built a script that needed to live on
unknown third party sites that may or may not already have another version of 
jQuery loaded. The jQuery packaged with my script is completely private to the 
js that requires it, eliminating the possibility of version conflict issues.

### The Power of Transforms

Before bundling your JavaScript, Browserify makes it easy for you to preprocess
your files through a[number of transforms][18] before including them in the
bundle. This is how you'd compile`.coffee` or `.hbs` files into your bundle as
valid JavaScript.

The most common way to do this is by listing your transforms in a 
`browserify.transform` object your `package.json` file. Browserify will apply
the transforms in the order in which they're listed. This assumes you've
`npm install`'d them already.

    "browserify": {
        "transform": ["coffeeify", "hbsfy" ]
      },
     "devDependencies": {
        "browserify": "~3.36.0",
        "coffeeify": "~0.6.0",
        "hbsfy": "~1.3.2",
        "gulp": "~3.6.0",
        "vinyl-source-stream": "~0.1.1"
      }
    

Notice that I've listed the transforms under `devDependencies` since they're
only used for preprocessing, and not in our final javascript output. You can do 
this automatically by adding the`--save-dev` or `-D` flag when you install.

    npm install someTransformModule --save-dev
    

Now we can `require('./view.coffee')` and `require('./template.hbs')` like we
would any other javascript file! We can also use the`extentions` option with
the Browserify API to tell browserify to recognize these extensions, so we don't
have to explicitly type them in our`require`s.

    browserify({
        entries: ['./src/javascript/app.coffee'],
        extensions: ['.coffee', '.hbs']
    })
    .bundle()
    ...
    

See this in action [here.][19]

## Using them together: Gulp + Browserify

Initially, I started out using the `gulp-browserify` plugin. A few weeks later
though,**Gulp added it to their [blacklist][20]**. Turns out the plugin was
unnecessary - you can can[node-browserify][21] API straight up, with a little
help from[`vinyl-source-stream`][22] This just converts the bundle into the
type of stream gulp is expecting. Using browserify directly is great because you
'll always have access to 100% of the features, as well as the most up-to-date 
version.

#### Basic Usage

    var browserify = require('browserify');
    var gulp = require('gulp');
    var source = require('vinyl-source-stream');
    
    gulp.task('browserify', function() {
        return browserify('./src/javascript/app.js')
            .bundle()
            //Pass desired output filename to vinyl-source-stream
            .pipe(source('bundle.js'))
            // Start piping stream to tasks!
            .pipe(gulp.dest('./build/'));
    });
    

#### Awesome Usage

Take a look at the [`browserify.js` task][19] and [package.json][23] in my
starter repo to see how to apply transforms for CoffeeScript and Handlebars, set
up non-common js modules and dependencies with[`browserify-shim`][24], and
handle compile errors through Notification Center. To Learn more about 
everything else you can do with Browserify,[read through the API][25]. I hope
you have as much fun with it as I'm having. Enjoy!

 [1]: http://www.100percentjs.com/just-like-grunt-gulp-browserify-now/
 [2]: https://github.com/greypants/gulp-starter
 [3]: https://github.com/greypants/gulp-starter/wiki
 [4]: http://nodejs.org/
 [5]: https://www.npmjs.org
 [6]: http://wiki.commonjs.org/wiki/Modules/1.1
 [7]: https://www.npmjs.org/doc/json.html
 [8]: http://gulpjs.com

 [9]: img/687474703a2f2f636c2e6c792f696d6167652f3266317533573347313030432f66696c652d7374727563747572652e706e67
 [10]: https://github.com/gulpjs/gulp/blob/master/docs/README.md
 [11]: http://browserify.org/
 [12]: http://requirejs.org
 [13]: https://github.com/jrburke/almond
 [14]: https://github.com/amdjs/amdjs-api/wiki/AMD
 [15]: http://nodejs.org/api/modules.html#modules_file_modules
 [16]: https://github.com/greypants/gulp-starter/wiki/What-is-npm%3F
 [17]: http://media.tumblr.com/tumblr_lm0h55WjoG1qdlkgg.gif
 [18]: https://github.com/substack/node-browserify/wiki/list-of-transforms

 [19]: https://github.com/greypants/gulp-starter/blob/master/gulp/tasks/browserify.js
 [20]: https://github.com/gulpjs/plugins/issues/47
 [21]: https://github.com/substack/node-browserify#api-example/node-browserify
 [22]: https://www.npmjs.org/package/vinyl-source-stream
 [23]: https://github.com/greypants/gulp-starter/blob/master/package.json
 [24]: https://github.com/thlorenz/browserify-shim
 [25]: https://github.com/substack/node-browserify#api-example
