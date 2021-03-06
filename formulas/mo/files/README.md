Mo - Mustache Templates in Bash
===============================

[Mustache] templates are simple, logic-less templates.  Because of their simplicity, they are able to be ported to many languages.  The syntax is quite simple.

    Hello, {{NAME}}.

    I hope your {{TIME_PERIOD}} was fun.

The above file is [`demo/fun-trip.mo`](demo/fun-trip.mo).  Let's try using this template some data from bash's environment.  Go to your checked out copy of the project and run a command like this:

    NAME=Tyler TIME_PERIOD=weekend ./mo demo/fun-trip.mo

Your result?

    Hello, Tyler.

    I hope your weekend was fun.

This bash version supports conditionals, functions (both as filters and as values), as well as indexed arrays (for iteration).  You are able to leverage these additional features by adding more information into the environment.  It is easiest to do this when you source `mo`.  See the [demo scripts](demo/) for further examples.


Requirements
------------

* Bash 3.x (the aim is to make it work on Macs)
* The "coreutils" package (`basename` and `cat`)
* ... that's it.  Why?  Because bash **can**!

If you intend to develop this and run the official specs, you also need node.js.


How to Use
----------

If you only plan using strings and numbers, nothing could be simpler.  In your shell script you can choose to export the variables.  The below script is [`demo/using-strings`](demo/using-strings).

    #!/usr/bin/env bash
    cd "$(dirname "$0")" # Go to the script's directory
    export TEST="This is a test"
    echo "Your message:  {{TEST}}" | ../mo

The result?  "Your message:  This is a test".

Using arrays adds a slight level of complexity.  *You must source `mo`.*  Look at [`demo/using-arrays`](demo/using-arrays).

    #!/usr/bin/env bash
    cd "$(dirname "$0")" # Go to the script's directory
    export ARRAY=( one two "three three three" four five )
    cat << EOF | . ../mo
    Here are the items in the array:
    {{#ARRAY}}
        * {{.}}
    {{/ARRAY}}
    EOF

The result?  You get a list of the five elements in the array.  Take a look at the line that executes `mo`.  You'll see that we went from `../mo` to using `. ../mo`.  That is very important when you want arrays to work, since you can not execute a command and have arrays passed to that command's environment.  Instead, we source the file.  `. ../mo` is identical to `source ../mo` and you can find more about that in bash's man page.

There are more scripts available in the [demos directory](demo/) that could help illustrate how you would use this program.


Concessions
-----------

I admit that implementing everything in bash just doesn't make a lot of sense.  For example, the following things just don't work because they don't really mesh with the "bash way".

Pull requests to solve the following issues would be helpful.


### Mustache Syntax

* Dotted names are not supported and this means associative arrays are not addressable via their index.  Partly this is because our target (Bash 3) does not support associative arrays.
* There's no "top level" object, so `echo '{.}' | ./mo` does not do anything useful.  In other languages you can say the data for the template is a string and in `mo` the data is always the environment.  Luckily this type of usage is rare and `{.}` works great when iterating over an array.
* HTML encoding is not built into `mo`.  `{{{var}}}`, `{{&var}}` and `{{var}}` all do the same thing.  `echo '{{TEST}}' | TEST='<b>' mo` will give you "`<b>`" instead of "`&gt;b&lt;`".
* You can not change the delimiters.


### General Scripting Issues

* Using binary files as templates is simply not allowed.
* Bash does not support nested structures like fancy objects.  The best you can do are arrays.  I'm not able to add super complex structures to bash - it's just a shell after all!
* You must make sure the data is in the environment when `mo` runs.  The easiest way to do that is to source `mo` in your shell script after setting up lots of other environment variables / functions.


Developing
----------

Check out the code and hack away.  Please add tests to show off bugs before fixing them.  New functionality should also be covered by a test.

To run against the official specs, you need to make sure you have the "spec" submodule.  If you see a `spec/` folder with stuff in it, you're already set.  Otherwise run `git submodule update --init`.  After that you need to install node.js and run `npm install async` (no, I didn't make a package.json to just list one dependency).  Finally, `./run-spec.js spec/specs/*.json` will run against the official tests - there's over 100 of them.


### Porting and Backporting

There is a diagnostic script that is aimed to help at making sure the internal functions all work when testing the script out on a version of bash or in an environment that is causing issues.  When adding new functions, please make sure this script gets updated to properly test your code.


### Failed Specs

It is acceptable for some of the official spec tests to fail.  Anything dealing with multiple levels of objects (eg. `{{a.b.c}}`) and changing the delimiters (`{{= | | =}}`) will fail.  Other than that, this bash implementation of the mustache spec should pass tests.

Specific issues:
 * Interpolation - Multiple Calls:  This fails because lambdas execute in a subshell so their output can be captured.  This is flagged as a TODO in the code.
 * HTML Escaping - Since bash is not often executed in a web server context, it makes no sense to have the output escaped as HTML.  Performing shell escaping of variables may be an option in the future if there's a demand.


### Future Enhancements

There's a few places in the code marked with `TODO` to signify areas that could use improvement.  Care to help?  Keep in mind that this uses bash exclusively, so it might not look the prettiest.


License
-------

This program is licensed under an MIT license with an additional non-advertising clause.  See [LICENSE.md](LICENSE.md) for the full text.


[Mustache]: https://mustache.github.io/
