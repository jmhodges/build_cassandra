
SOY = ENV['USE_64_BIT_JVM'] ? 'soylatte16-amd64-1.0.3' : 'soylatte16-i386-1.0.3'

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

task :default => :start

desc "Start Cassandra server"
task :start do
  cd here('cassandra')
  jvm_sh 'bin/cassandra -f'
end

desc "Start Cassandra command line interface"
task :cli do
  cd here('cassandra')
  jvm_sh 'bin/cassandra-cli'
end

desc "Compile Cassandra"
task :compile do
  cd here('cassandra')
  jvm_sh("ant")
  mkdir_p(here('data'))
  Dir['conf/*'].each do |fn|
    fr = File.open(fn).read.split("\n")
    fr.each{|l| l.gsub!(%r{/var/cassandra}, here('data')) }
    File.open(fn, 'w'){|f| f.write(fr.join("\n"))}
  end
end

desc "Clean current Cassandra build"
task :clean do
  cd here('cassandra')
  jvm_sh("ant clean")
end

task :rebuild => [:clean, :compile]

task :clobber => :clean do
  rm_rf here('data/*')
end

desc "Pass the contents of the 'doit' ENV string to the Java 1.7 runtime"
task :jvm do
  jvm_sh "java -version"
  jvm_sh ENV['doit'] if ENV['doit']
end

desc "Setup all dependencies"
task :setup => [:icedtea, :soylatte, :bsdport, :cassandra_source]

desc "Checkout or update the Cassandra source code"
task :cassandra_source => :svn do
  if File.exist?(here('cassandra'))
    scm_command = 'svn up'
    if File.exist?(here('cassandra/.git'))
      scm_command = 'git svn rebase'
    end
    cd here('cassandra') do
      sh(scm_command)
    end

  else
    sh("svn co https://svn.eu.apache.org/repos/asf/incubator/cassandra/trunk cassandra")
  end
end

task :soylatte do
  tarball = "#{SOY}.tar.bz2"
  unless File.exist?(SOY)
    unless ENV['I_AGREE_WITH_THE_SOYLATTE_LICENSE']
      puts "You must agree to the Soylatte license to continue. Go read it:"
      puts "http://landonf.bikemonkey.org/static/soylatte/#get"
      puts "Then pass I_AGREE_WITH_THE_SOYLATTE_LICENSE=1 as an environment variable."
      exit(1)
    end  
    sh "curl 'http://jrl:I am a Licensee in good standing@hg.bikemonkey.org/archive/javasrc_1_6_jrl_darwin/#{tarball}' -o #{tarball}"
    sh "tar xjvf #{tarball}"
  end
end

task :icedtea do
  icedtea = 'jdk-7-icedtea-plugs'
  tarball = "#{icedtea}-1.6.tar.gz"
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
   ALT_BOOTDIR=#{here(SOY)} \
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
  cd here('bsd-port')
  sh "sh build.sh"
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
