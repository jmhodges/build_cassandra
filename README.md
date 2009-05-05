# Building and Running Cassandra on Mac OS X

So, [Cassandra](http://incubator.apache.org/projects/cassandra.html)
is pretty awesome. And Cassandra depends on Java 7. Sort of.

What's really going on is that you can use Cassandra on Java 6, but
the jar files included are often built by the devs against 1.7. Also,
apparently on quad core machines, Java 6 will spontaneously crash, so
the Facebook guys are like "oh, hell, no". In any case, not using Java
7 for Cassandra is harder than using it.

Except for building Java 7 on OS X. That's a total bitch.

Oh, and running it without having to use a very beta version of
Java. That's a total bitch, too.

But, you know, we're software developers. We write Solutions For The
Enterprise<sup>TM</sup>, and Solutions For This Horrible
Itch In The Middle Of Our Backs and also Solutions To Stop Having to
Friggin' Do This Goddamn It Is Annoying. So, I wrote this thing.

It requires nothing more than Ruby, Rake, and MacPorts plus the usual
gcc/make/ant stuff.

### NOTE: This does not have Java 7 infecting your machine! It doesn't
### change JAVA_HOME or PATH.

## Building Cassandra

Take a look at the Rakefile here. You'll need to change the line
`I_AM_OKAY_WITH_SOYLATTE_LICENSE = false` line to
`I_AM_OKAY_WITH_SOYLATTE_LICENSE = true` if you are okay with the
license described at the [soylatte
site](http://landonf.bikemonkey.org/static/soylatte/#get).

After that, run
    
    rake setup compile

You'll have to type in your password to sudo to install mercurial via
MacPorts but other than that, you're done. `setup` will build the Java
1.7 stuff and `compile` will build Cassandra against that new
Java.

It'll even make a data directory here and edit the config files to use
it as the log and data directory for Cassandra.

No config, no hassle. Isn't that awesome?

(Oh, and by the way, it edits your `~/.hgrc` file to add the
`forest-crew` extension because Mercurial a) sucks and b) blows and,
apparently, people feel the need to require you to use horrible things
that require bolted on libraries to work properly. Feel free to remove
it or the whole file after you're done. The code does a good job of
not screwing the pooch in your `.hgrc`, so don't fret about it
changing.)

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

That runs the command set in doit with the `JAVA_HOME` and `PATH` set
appropriately.

    rake clobber

This cleans out the Cassandra build, and removes the data stuff under
the `data` directory.

    rake rebuild

This runs `clean` and `compile` for you.
