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
  cd here('cassandra') do
    jvm_sh 'bin/cassandra -f'
  end
end

desc "Start Cassandra command line interface"
task :cli do
  cd here('cassandra') do
    jvm_sh 'bin/cassandra-cli'
  end
end

desc "Compile Cassandra"
task :compile do
  cd here('cassandra') do
    jvm_sh("ant")
    mkdir_p(here('data'))
    Dir['conf/*'].each do |fn|
      fr = File.open(fn).read.split("\n")
      fr.each{|l| l.gsub!(%r{/var/cassandra}, here('data')) }
      File.open(fn, 'w'){|f| f.write(fr.join("\n"))}
    end
  end
end

desc "Clean current Cassandra build"
task :clean do
  cd here('cassandra') do
    jvm_sh("ant clean")
  end
end

task :rebuild => [:clean, :compile]

task :clobber => :clean do
  rm_rf here('data/*')
end

desc "Pass the contents of the 'doit' ENV string to the Java 1.7 runtime"
task :jvm do
  cd Rake.original_dir do
    jvm_sh ENV['doit'] if ENV['doit']
  end
end

desc "Setup all dependencies"
task :setup => [:soylatte, :icedtea, :bsdport, :cassandra]

desc "Checkout or update the Cassandra source code"
task :cassandra => :svn do
  if File.exist?(here('cassandra/.svn'))
    scm_command = 'svn up'
  elsif !File.exist?(here('cassandra'))
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
  cd here('bsd-port') do
    sh "sh build.sh"
  end
end

task :merc do
  sh('which hg') do |ok, res|
    if ! ok
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
  sh('which svn') do |ok, res|
    if ! ok
      sh 'sudo port install subversion +bash_completion'
    end
  end
end

# desc "Generate code from Cassandra's Thrift file"
task :gen => :thrift do
  sane = {'ruby' => 'rb', 'python' => 'py', 'c++' => 'cpp', 'smalltalk' => 'st'}
  if ENV['GEN'].nil?
    puts "\n\nYou forget to pass in some languages in GEN. As in, GEN='ruby py', etc."
    exit(1)
  end
  
  gen = ENV['GEN'].split.map{|l| sane[l] || l}.join(" ")
  mkdir_p here('generated')
  sh "thrift  -o #{here('generated')} --gen #{gen} #{here('cassandra/interface/cassandra.thrift')}"
  puts "See the 'generated' directory for the generated code"
end

task :thrift => [:boost, :flex, :bison] do
  sh('which thrift') do |ok, res|
    if ! ok
      two_weeks_ago = Time.now - ( 86400 * 14 )
      tarball = here('thrift.tgz')
      if !File.exist?(here('thrift.tgz')) || File.ctime(here('thrift.tgz')) < two_weeks_ago
        sh "curl 'http://gitweb.thrift-rpc.org/?p=thrift.git;a=snapshot;h=HEAD;sf=tgz' -o #{tarball}"
      end
      sh "tar xzf #{tarball}"

      cd 'thrift' do
        sh '/bin/bash ./bootstrap.sh'
        # Using jvm_sh so we get the right javac for generation
        jvm_sh "CONFIG_SHELL=/bin/bash ./configure"
        sh 'make'
        sh 'sudo make install'
      end
    end
  end
end

task :boost do
  sh('which bjam') do |ok, res|
    if ! ok
      sh('sudo port install boost')
    end
  end
end

task :flex do
  sh('which flex') do |ok, res|
    if ! ok
      sh('sudo port install flex')
    end
  end
end

task :bison do
  sh('which bison') do |ok, res|
    if ! ok
      sh('sudo port install bison')
    end
  end
end
