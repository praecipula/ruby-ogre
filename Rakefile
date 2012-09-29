require 'rubygems/package_task'
require 'rake/clean'

#
# Download sources.
#
namespace :download do

  task :ois do
    FileUtils::mkdir_p("deps/src")
    chdir('deps/src') {
      unless /mingw/ =~ RUBY_PLATFORM
        OIS_FILE="ois_v1-3.tar.gz"
        sh "wget http://downloads.sourceforge.net/project/wgois/Source%20Release/1.3/#{OIS_FILE}"
        sh "tar xzvf #{OIS_FILE}"
      end
    }
  end
  
  task :ogre do
    FileUtils::mkdir_p("deps/src")
    chdir('deps/src') {
      if /mingw/ =~ RUBY_PLATFORM
        sh "wget http://sourceforge.net/projects/ogre/files/ogre/1.8/1.8.0/OgreSDK_MinGW_v1-8-0.exe"
        sh "OgreSDK_MinGW_v1-8-0.exe"
        sh "cp -a OgreSDK_MinGW_v1-8-0/include ../"
        sh "cp -a OgreSDK_MinGW_v1-8-0/boost/boost ../include"
        sh "cp -a OgreSDK_MinGW_v1-8-0/lib ../"
        sh "cp -a OgreSDK_MinGW_v1-8-0/boost/lib/* ../lib"
      else
        OGRE_FILE="ogre_src_v1-8-0.tar.bz2"
        sh "wget http://sourceforge.net/projects/ogre/files/ogre/1.8/1.8.0/#{OGRE_FILE}"
        sh "tar xjvf #{OGRE_FILE}"
      end
    }
  end

  task :procedural do
    FileUtils::mkdir_p("deps/src")
    chdir('deps/src') {
      if /mingw/ =~ RUBY_PLATFORM
        sh "wget http://ogre-procedural.googlecode.com/files/OgreProceduralSDK_MingW_v0.2.7z"
        sh "C:\Program Files (x86)\7-Zip\7z.exe x OgreProceduralSDK_MingW_v0.2.7z"
        FileUtils::mkdir_p("./")
        sh "cp -a OgreProceduralSDK_MingW_v0.2/include ../"
        sh "cp -a OgreProceduralSDK_MingW_v0.2/lib ../"
      else
        sh "hg clone -b v0-2 https://code.google.com/p/ogre-procedural/ ogre-procedural"
      end
    }
  end

  task :all => [:ois, :ogre, :procedural]
end

desc "Download everything"
task :download => 'download:all'


namespace :compile do

  #
  # Compile libraries.
  #
  namespace :deps do

    task :ois do
      unless /mingw/ =~ RUBY_PLATFORM
        chdir("deps/src/ois-v1-3/") {
          sh "sh ./bootstrap"
          sh "./configure --prefix=$PWD/../.."
          sh "make && make install"
        }
      end
    end

    task :ogre do
      unless /mingw/ =~ RUBY_PLATFORM
        chdir("deps/src/ogre_src_v1-8-0/") {
          sh "cmake -DCMAKE_INSTALL_PREFIX:PATH=../.. -DOGRE_INSTALL_SAMPLES:BOOL=ON -DOIS_INCLUDE_DIR:PATH=$PWD/../../include/OIS -DOIS_LIBRARY_DBG:FILEPATH=$PWD/../../lib/libOIS.so -DOIS_LIBRARY_REL:FILEPATH=$PWD/../../lib/libOIS.so -DCMAKE_MODULE_LINKER_FLAGS:STRING='-static-libgcc -static-libstdc++' -DCMAKE_SHARED_LINKER_FLAGS:STRING='-static-libgcc -static-libstdc++'"
          sh "make -j4 && make install"
        }
      end
    end

    task :procedural do
      unless /mingw/ =~ RUBY_PLATFORM
        chdir("deps/src/ogre-procedural/") {
          sh "OGRE_HOME=../..; OIS_HOME=../..; cmake -DCMAKE_INSTALL_PREFIX:PATH=../.. -DOgreProcedural_BUILD_SAMPLES:BOOL=ON"
          sh "make -j4 && make -i install"
        }
      end
    end

    #
    # Copy the compiled dynamic libraries of Ogre.
    #
    task :copylibs do
      if /mingw/ =~ RUBY_PLATFORM
        FileUtils::mkdir_p("./deps/lib/OGRE")
        sh "cp -a deps/src/OgreSDK_MinGW_v1-8-0/bin/release/*.dll deps/lib/"
        sh "cp -a deps/src/OgreSDK_MinGW_v1-8-0/bin/release/Plugin*.dll deps/lib/OGRE"
        sh "cp -a deps/src/OgreSDK_MinGW_v1-8-0/bin/release/RenderSystem*.dll deps/lib/OGRE"
        sh "cp -a deps/src/OgreProceduralSDK_MingW_v0.2/bin/Release/*.dll deps/lib/"
        FileUtils::mkdir_p("./deps/share/OGRE")
        sh "cp -a deps/src/OgreSDK_MinGW_v1-8-0/media deps/share/OGRE"
      end 
    end
    
    task :all => [:ois, :ogre, :procedural, :copylibs]
  end

  desc "Compile all of deps libraries."
  task :deps => 'deps:all'


  #
  # Compile extensions
  #
  namespace :ext do
    DLEXT = RbConfig::MAKEFILE_CONFIG['DLEXT']

    task :all => ["lib/ois.#{DLEXT}", "lib/ogre.#{DLEXT}", "lib/ogrebites.#{DLEXT}", "lib/procedural.#{DLEXT}"] 

    ## lib/*.#{DLEXT}
    file "lib/ois.#{DLEXT}" => "bindings/ois/ois.#{DLEXT}" do |f|
      cp f.prerequisites, "lib/", :preserve => true
    end
    
    file "lib/ogre.#{DLEXT}" => "bindings/ogre/ogre.#{DLEXT}" do |f|
      cp f.prerequisites, "lib/", :preserve => true
    end
    
    file "lib/ogrebites.#{DLEXT}" => "bindings/ogrebites/ogrebites.#{DLEXT}" do |f|
      cp f.prerequisites, "lib/", :preserve => true
    end
    
    file "lib/procedural.#{DLEXT}" => "bindings/procedural/procedural.#{DLEXT}" do |f|
      cp f.prerequisites, "lib/", :preserve => true
    end
    
    ## ext/**/*.#{DLEXT}
    file "bindings/ois/ois.#{DLEXT}" => FileList["bindings/ois/Makefile"] do |f|
      sh 'cd bindings/ois/ && make clean && make'
    end
    CLEAN.include 'bindings/ois/*.{o,so,dll}'
    
    file "bindings/ogre/ogre.#{DLEXT}" => FileList["bindings/ogre/Makefile"] do |f|
      sh 'cd bindings/ogre/ && make clean && make'
    end
    CLEAN.include 'bindings/ogre/*.{o,so,dll}'
    
    file "bindings/ogrebites/ogrebites.#{DLEXT}" => FileList["bindings/ogrebites/Makefile"] do |f|
      sh 'cd bindings/ogrebites && make clean && make'
    end
    CLEAN.include 'bindings/ogrebites/*.{o,so,dll}'
    
    file "bindings/procedural/procedural.#{DLEXT}" => FileList["bindings/procedural/Makefile"] do |f|
      sh 'cd bindings/procedural/ && make clean && make'
    end
    CLEAN.include 'bindings/procedural/*.{o,so,dll}'

    ## ext/**/Makefile
    file 'bindings/ois/Makefile' => FileList['bindings/ois/interface/ois_wrap.cpp'] do
      chdir('bindings/ois/') { ruby 'extconf.rb' }
    end
    CLEAN.include 'bindings/ois/Makefile'
    
    file 'bindings/ogre/Makefile' => FileList['bindings/ogre/interface/ogre_wrap.cpp'] do
      chdir('bindings/ogre/') { ruby 'extconf.rb' }
    end
    CLEAN.include 'bindings/ogre/Makefile'
    
    file 'bindings/ogrebites/Makefile' => FileList['bindings/ogrebites/interface/ogrebites_wrap.cpp'] do
      chdir('bindings/ogrebites/') { ruby 'extconf.rb' }
    end
    CLEAN.include 'bindings/ogrebites/Makefile'
    
    file 'bindings/procedural/Makefile' => FileList['bindings/procedural/interface/procedural_wrap.cpp'] do
      chdir('bindings/procedural/') { ruby 'extconf.rb' }
    end
    CLEAN.include 'bindings/procedural/Makefile'
    
    ## make wrappers with swig.
    file 'bindings/ois/interface/ois_wrap.cpp' do
      chdir('bindings/ois/interface') { sh 'rake' }
    end
    CLEAN.include 'bindings/ois/interface/ois_wrap.{cpp,h,o}'
    
    file 'bindings/ogre/interface/ogre_wrap.cpp' do
      chdir('bindings/ogre/interface') { sh 'rake' }
    end
    CLEAN.include 'bindings/ogre/interface/ogre_wrap.{cpp,h,o}'
    
    file 'bindings/ogrebites/interface/ogrebites_wrap.cpp' do
      chdir('bindings/ogrebites/interface') { sh 'rake' }
    end
    CLEAN.include 'bindings/ogrebites/interface/ogrebites_wrap.{cpp,h,o}'
    
    file 'bindings/procedural/interface/procedural_wrap.cpp' do
      chdir('bindings/procedural/interface') { sh 'rake' }
    end
    CLEAN.include 'bindings/procedural/interface/procedural_wrap.{cpp,h,o}'
  end

  desc "Compile all of extension libraries."
  task :ext => 'ext:all'
end


#
# Document
#
desc 'Create documents'
task :doc => ['bindings/ois/interface/ois_wrap.cpp',
              'bindings/ogre/interface/ogre_wrap.cpp',
              'bindings/ogrebites/interface/ogrebites_wrap.cpp',
              'bindings/procedural/interface/procedural_wrap.cpp'] do |f|

  sh "rdoc #{f.prerequisites.join(' ')}"
end

#
# Gemspec
#
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'version'

spec = Gem::Specification.new do |s|

  s.name        = "ruby-ogre"
  s.version     = Ruby::Ogre::VERSION
  s.summary     = "A ruby wrapper set of Ogre, OIS and Procedural."
  s.homepage    = "https://github.com/abexsoft/ruby-ogre"
  s.authors     = ["abexsoft works"]
  s.email       = ["abexsoft@gmail.com"]
  s.description = "A ruby wrapper set of Ogre, OIS and Procedural."
  s.platform    = Gem::Platform::CURRENT
 
  # The list of files to be contained in the gem 
  s.files = FileList['Rakefile',
                     'README.md',
                     'INSTALL.md',
                     'LICENSE',
                     'doc/**/*',
                     'bindings/ogre/interface/**/*',
                     'bindings/ogrebites/interface/**/*',
                     'bindings/ois/interface/**/*',
                     'bindings/procedural/interface/**/*',
                     'lib/**/*',
                     'deps/lib/libOIS*.so*',
                     'deps/lib/libOgre*.so*',
                     'deps/lib/*.dll',
                     'deps/lib/OGRE/*.so*',
                     'deps/lib/OGRE/*.dll',
                     'deps/include/**/*',
                     'deps/share/OGRE/media/**/*',
                     'sample/charactor/*.rb',
                     'sample/charactor/sinbad*',
                     'sample/charactor/resources.cfg',
                     'sample/charactor/plugins.cfg'
                    ].exclude('deps/share/OGRE/media/Makefile',
                              'deps/share/OGRE/media/CMakeFiles/**/*',
                              'deps/share/OGRE/media/cmake_install.cmake').to_a

  s.require_paths = ["lib"]
end

Gem::PackageTask.new(spec) do |pkg|
end
