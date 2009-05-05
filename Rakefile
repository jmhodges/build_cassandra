 # edit this to true if you are okay with the soylatte license. go
 # read the license
 # wankers. http://landonf.bikemonkey.org/static/soylatte/#get
I_AM_OKAY_WITH_SOYLATTE_LICENSE = false

require 'rubygems'
require 'rake'
require 'uri'


def here(filename)
  File.expand_path(File.dirname(__FILE__) + '/' + filename)
end

JAVA17_SDK = here 'bsd-port/build/bsd-i586/j2sdk-image/'

def jvm_sh(command)
  sh "JAVA_HOME=#{JAVA17_SDK} PATH=#{JAVA17_SDK}/bin:$PATH " + command
end

task :default do
  puts "Run 'rake setup compile' if this is your first time with this app. If you haven't edited the Rakefile to be okay with the license agreement, do so now. Just change the line 'I_AM_OKAY_WITH_SOYLATTE_LICENSE = false' to 'I_AM_OKAY_WITH_SOYLATTE_LICENSE = true'"
  puts "Run 'rake start' if you just want to boot Cassandra"
end

task :start do
  cd here('cassandra')
  jvm_sh 'bin/cassandra -f'
end

task :cli do
  cd here('cassandra')
  jvm_sh 'bin/cassandra-cli'
end

task :compile do
  cd here('cassandra')
  jvm_sh("ant")
  mkdir_p(here('data'))
  Dir['conf/*'].each do |fn|
    fr = File.open(fn).read.split("\n")
    fr.each{|l| l.gsub!(%r{/var/cassandra}, here('data')) }
    File.open(fn, 'w').write(fr.join("\n"))
  end
end

task :clean do
  cd here('cassandra')
  jvm_sh("ant clean")
end

task :rebuild => [:clean, :compile]

task :clobber => :clean do
  rm_rf here('data/*')
end

task :jvm do
  jvm_sh "java -version"
  jvm_sh ENV['doit'] if ENV['doit']
end

task :setup => [:cassandra_source, :bsdport, :icedtea, :soylatte] do
  cd here('bsd-port')
  sh "make clean"
  sh "sh build.sh"
end

task :cassandra_source => :svn do
  unless File.exist?(here('cassandra'))
    sh('svn co https://svn.eu.apache.org/repos/asf/incubator/cassandra/trunk cassandra')
  end
end
SOY = 'soylatte16-i386-1.0.3'
task :soylatte do
  soy = SOY
  tarball =  "#{soy}.tar.bz2"
  if I_AM_OKAY_WITH_SOYLATTE_LICENSE
    pass = "jrl:I am a Licensee in good standing@"
  else
    puts "YOU didn't change I_AM_OKAY_WITH_SOYLATTE_LICENSE in the Rakefile so now I'm exiting because rargh!"
    exit(1)
  end
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
   ALT_BOOTDIR=#{SOY} \
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
    if ! ok && !(sh('which hg') rescue false)
      sh 'sudo port install mercurial +bash_completion'
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
    File.open(hgrc_path, 'w'){|f| f.write(new_hgrc.join("\n"))}
  end
end

task :svn do
  sh('port installed subversion | grep active') do |ok, res|
    if ! ok && !(sh('which svn') rescue false)
      sh 'sudo port install subversion +bash_completion'
    end
  end
end
