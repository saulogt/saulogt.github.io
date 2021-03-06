---
layout: post
title:  "Migrating every Javascript to Typescript code"
date:   2016-01-09 10:00:00
categories: []
tags: [ javascript, typescript, nodejs, programming ]
image: "/images/2016/js2ts.png"
published: true
---

![js2ts](/images/2016/js2ts.png)

A really talented coworker of mine tried to convince me to use CoffeeScript some time ago, but with no success,
even showing me the amazing work he was doing with it. See his [GitHub profile](https://github.com/jaykon-w).  
The problem that I found then, was that the code I had to write was sort of far from plain javascript (IMO). I admit. I'm not a good JS developer. My background is C++, but I really enjoy learning "new" stuff.

Another excuse that I usually hear (from myself as well) is: "I don't like the magic code generated by this tool.
I don't want to lose control of my code". Now I know that it doesn't make sense.
Typescript is more about "compilation time" checks and the code generated is almost
 the same you would write in pure js (At least the way I do it).

For almost an year I abandoned the idea of using anything but vanilla JS.  

After watching a video about TypeScript I decided to give a chance to another transpiler, now with much more buzz around the tech scene.

The steps I took to migrate were:

###1. Rename my *.js files to *.ts.

Only this was responsible find some typos in a development code, as you can see below

{% highlight javascript %}

var myVariable = false;

function changeState(){
    myVariable = !myvariable
};

{% endhighlight %}

You might not have noticed, but typescript found a bug here with the following message:


```>> filters/labelTranslator.ts(61,19): error TS2304: Cannot find name 'myvariable'.```


###2. Adapt Gruntfile.js to include the transpilation step.

This is crucial if you want to be or if you are professional!
Yes, I know that some professional don't use automation tools like grunt or gulp. Trust me... using typescript combined with a proper workflow, will ensure much higher quality of your releases.

First I installed grunt-typescript with the command line ```npm install grunt-typescript --save-dev```

Then I configured the step like this:

{% highlight javascript %}
grunt.loadNpmTasks('grunt-typescript');

    grunt.initConfig({
      ...
      typescript: {
        base: {
          src: ['<%= yeoman.app %>/scripts/{,**/}*.ts'],
          options: {
            module: 'amd', //or commonjs
            target: 'es5', //or es3
            sourceMap: true, // This line allows the bugging through map files
            declaration: true
          }
        }
      },
      ...
    });
{% endhighlight %}
This configuration will create the js files in the same folder where the ts files are.

And finally I configured the tasks including the ```'typescript'``` step where suitable.

###3. Start using types

Here is where typescript really shines.

Consider the following code:

{% highlight javascript %}
function greeter(person) {
    return "Hello, " + person;
}

var user = "Saulo Tauil";

alert(greeter(user))
{% endhighlight %}

What if I set ```user``` to ```true```? Yeah it should not allow it. That's why I can do typescript like this:
{% highlight javascript %}
function greeter(person: string) {
    return "Hello, " + person;
}
{% endhighlight %}

See what happens now:

![typescriptconplaining](/images/2016/typescript_complaininf_of_string.png)

just with this simple type declaration, typescript prevent me to brake my own code.

###Conclusion

Javascript is really nice. Even more now that NodeJS became so popular.

However, a lot of senior developers and managers complain that it is not reliable when dealing with huge systems, many developers, frequent commit and so on.
Using TypeScriot we can ensure the same minimal check as compiled language such as Java and C++. In fact it is possible to set the typescript transpiler to deny automatic any type and force type definition everywehre.
