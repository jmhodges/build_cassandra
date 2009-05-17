# Building and Running Cassandra on Mac OS X

So, [Cassandra](http://incubator.apache.org/cassandra/) is pretty
awesome. If you've found this, you've probably read that Cassandra
requires Java 7 because of jars it depends on. This is no
longer true. However, this tool still makes it damn easy to build
Cassandra against Java 7 and if you want or need it, I'd recommend using
this Rakefile.

You see, building Java 7 on OS X is a total bitch. Oh, and running it
without having to use a very beta version of Java for everything on
your machine. That's a total bitch, too.

But, you know, we're software developers. We write Solutions For The
Enterprise<sup>TM</sup>, and Solutions For This Horrible
Itch In The Middle Of Our Backs and also Solutions To Stop Having to
Friggin' Do This Goddamn It Is Annoying. So, I wrote this thing.

It requires nothing more than Ruby, Rake, and MacPorts plus the usual
gcc/make/ant stuff.

### NOTE:

This program is awesome because it handles the `JAVA_HOME` and `PATH`
stuff for building and running Cassandra without adding Java 7 to them
permanently. You won't have Java 7 all up in your business after you
run this.

## Building Cassandra

Before compiling you'll need to approve of the SoyLatte license. To
agree to the license (described at the [SoyLatte
site](http://landonf.bikemonkey.org/static/soylatte/#get)), you will
need to put `I_AGREE_WITH_THE_SOYLATTE_LICENSE=1` at the beginning
of the the setup command I show you below.

Since you've totally read these docs up to this point, you know you
need to prepend that environment variable to this command:
    
    rake setup compile

You'll have to type in your password to sudo to install Mercurial and
Subversion via MacPorts (if you don't already have a `hg` and `svn`
binary, respectively) but other than that, you're done.

`setup` will build the Java 1.7 stuff and `compile` will build
Cassandra against that new Java. It'll even make a data directory here
and edit the config files to use it as the log and data directory for
Cassandra.

No config, no hassle. Isn't that awesome?

If you have problems or want details, check out the Extra Notes on
Building Section below.

## Running Cassandra

You can start up Cassandra with the command

    rake
or

    rake start

That boots up Cassandra in the foreground. You can get to the nice
command line interface with

    rake cli

Why use these instead of straight `./bin/cassandra -f` and so on?
Because you don't want to edit your `JAVA_HOME` and `PATH` to include this
beta Java install and those problems are taken care of when using this
Rakefile.

## Bonus round

    rake jvm doit="some-java-command"

That runs the command set to `doit` with the `JAVA_HOME` and `PATH` set
appropriately.

    rake clean

This just runs `ant clean` in the Cassandra codebase.

    rake clobber

This cleans out the Cassandra build via `ant clean`, and removes the data stuff under
the `data` directory.

    rake rebuild

This runs `clean` and `compile` for you.

## Extra Notes On Building

If you want the 64-bit AMD arch version of SoyLatte, add
`USE_64_BIT_JVM=1` to the beginning of your `rake setup` call.

Also, this Rakefile edits your `~/.hgrc` file to add the
`forest-crew` extension because Mercurial a) sucks and b) blows and,
apparently, people feel the need to require you to use horrible things
that require bolted on libraries to work properly. Feel free to remove
it or the whole file after you're done. The code does a good job of
not screwing the pooch in your `.hgrc`, so don't fret about it
changing.

### Possible Problems

If you're having a problem that's talking jive about `{standard
input}:731:junk \`f' after expression` or similar, you need to update
your version of Xcode to something much more recent.

And if you're having a problem with iconv, you're probably running a
MacPort version of it that is too new. Uninstall it, get the build of
SoyLatte to work (i.e. `rake setup` finishes perfectly), then you can
reinstall it.

## License

(The MIT License)

Copyright (c) 2009 Jeff Hodges

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
