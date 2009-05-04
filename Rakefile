require 'rubygems'
require 'rake'
require 'uri'
# Why aren't we using buildr? Because we have to start up a JVM that
# we don't necessarily want a jruby built against. Specifically, java
# 1.7.

def here(filename)
  File.expand_path(File.dirname(__FILE__) + '/' + filename)
end

BSDPORT_SDK = here 'bsd-port/build/bsd-i586/j2sdk-image/'

def jvm_sh(command)
  sh "JAVA_HOME=#{BSDPORT_SDK} PATH=#{BSDPORT_SDK}/bin:$PATH " + command
end

task :default => :compile

task :compile do
  cd here('cassandra')
  jvm_sh("ant")
end

task :clean do
  cd here('cassandra')
  jvm_sh("ant clean")
end

task :rebuild => [:clean, :compile]

task :jvm do
  jvm_sh "java -version"
  jvm_sh ENV['doit'] if ENV['doit']
end

task :setup => [:bsdport, :icedtea, :soylatte] do
  cd here('bsd-port')
  sh "make clean"
  sh "sh build.sh"
end

task :soylatte do
  soy = 'soylatte16-i386-1.0.3'
  tarball =  "#{soy}.tar.bz2"
  # pass = "jrl:I am a Licensee in good standing@"
  puts "YOU MUST HIT THAT LICENSE. I AM NOT DOING ANYTHING ILLEGAL ON YOUR BEHALF"
  exit(1)
  unless File.exist? soy
    sh "curl 'http://#{pass}hg.bikemonkey.org/archive/javasrc_1_6_jrl_darwin/#{tarball}' -o #{tarball}"
    sh "tar xjvf #{tarball}"
  end
end

task :icedtea do
  icedtea = 'jdk-7-icedtea-plugs-1.6'
  tarball = "#{icedtea}.tar.gz"
  unless File.exist?(icedtea)
    sh "curl http://landonf.bikemonkey.org/static/soylatte/#{tarball} -o #{tarball}"
    sh "tar xzvf #{tarball}"
  end
end

task :bsdport => :merc do
  unless File.exist? here('bsd-port')
    sh 'hg fclone http://hg.openjdk.java.net/bsd-port/bsd-port'
  end

  File.open(here('bsd-port/build.sh'), 'w') do |f|
    f.write(<<EOS
   LC_ALL=C
   LANG=C
   unset CLASSPATH
   unset JAVA_HOME
   make \
   ALT_BOOTDIR=#{SOYLATTE} \
   ALT_BINARY_PLUGS_PATH=#{here('jdk-7-icedtea-plugs')} \
   ALT_FREETYPE_HEADERS_PATH=/usr/X11R6/include \
   ALT_FREETYPE_LIB_PATH=/usr/X11R6/lib \
   ALT_CUPS_HEADERS_PATH=/usr/include \
   ANT_HOME=/usr/share/ant \
   NO_DOCS=true \
   HOTSPOT_BUILD_JOBS=1
EOS
            )
  end
end

task :merc do
  sh('port installed mercurial | grep active') do |ok, res|
    if ! ok
      sudo 'port install mercurial +bash_completion'
    end
  end

  hgrc_path = ENV['HOME'] + '/.hgrc'
  if File.exist?(hgrc_path)
    hgrc = File.open(hgrc_path).read.split("\n")
  else
    hgrc = []
  end
  
  unless hgrc.any?{|l| l =~ /^hgrc\.forest/ }
    sh 'hg clone http://bitbucket.org/pmezard/hgforest-crew'

    new_hgrc = []
    hgrc << '[extensions]' unless hgrc.any?{|l| l =~ /^\[extensions\]/ }
    hgrc.each do |l|
      new_hgrc << l
      if l =~ /^\[extensions\]/
        new_hgrc << "hgrc.forest="+here('hgforest-crew')+"/forest.py\n"
      end
    end
    File.open(hgrc_path, 'w').write(new_hgrc.join("\n"))
  end
end
