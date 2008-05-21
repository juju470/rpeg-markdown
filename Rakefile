require 'rake/clean'
require 'rake/packagetask'
require 'rake/gempackagetask'

DLEXT = Config::CONFIG['DLEXT']
VERS = '0.1.0'

spec =
  Gem::Specification.new do |s|
    s.name              = "rpeg-markdown"
    s.version           = VERS
    s.summary           = "Ruby extension library for peg-markdown"
    s.files             = FileList['README','LICENSE','Rakefile','test.rb','{lib,ext}/**.rb','ext/*.{c,h}','bin/rpeg-markdown']
    s.bindir            = 'bin'
    s.executables       << 'rpeg-markdown'
    s.require_path      = 'lib'
    s.has_rdoc          = true
    s.extra_rdoc_files  = ['README', 'LICENSE']
    s.test_files        = Dir['test.rb']
    s.extensions        = ['ext/extconf.rb']

    s.author            = 'Ryan Tomayko'
    s.email             = 'r@tomayko.com'
    s.homepage          = 'http://github.com/rtomayko/rpeg-markdown'
    s.rubyforge_project = 'wink'
  end

  Rake::GemPackageTask.new(spec) do |p|
    p.gem_spec = spec
    p.need_tar_gz = true
    p.need_tar = false
    p.need_zip = false
  end

namespace :submodule do
  desc 'Init the peg-markdown submodule'
  task :init do |t|
    unless File.exist? 'peg-markdown/markdown.c'
      rm_rf 'peg-markdown'
      sh 'git submodule init peg-markdown'
      sh 'git submodule update peg-markdown'
    end
  end

  task :update => :init do
    sh 'git submodule update peg-markdown'
  end
end

desc 'Gather required peg-markdown sources into extension directory'
task :gather => 'submodule:update' do |t|
  sh 'cd peg-markdown && make markdown_parser.c'
  cp FileList['peg-markdown/markdown_{peg.h,parser.c,output.c}'], 'ext/',
    :preserve => true,
    :verbose => true
end
CLOBBER.include 'ext/markdown_{peg.h,parser.c,output.c}'

file 'ext/Makefile' => FileList['ext/{extconf.rb,*.c,*.h,*.rb}'] do
  chdir('ext') { ruby 'extconf.rb' }
end
CLEAN.include 'ext/Makefile'

file "ext/markdown.#{DLEXT}" => FileList['ext/Makefile', 'ext/*.{c,h,rb}'] do |f|
  sh 'cd ext && make'
end
CLEAN.include 'ext/*.{o,bundle,so}'

file "lib/markdown.#{DLEXT}" => "ext/markdown.#{DLEXT}" do |f|
  cp f.prerequisites, "lib/", :preserve => true
end

desc 'Build the peg-markdown extension'
task :build => "lib/markdown.#{DLEXT}"

desc 'Run unit tests'
task 'test:unit' => [ :build ] do |t|
  ruby 'test.rb'
end

desc 'Run conformance tests'
task 'test:conformance' => [ 'submodule:update', :build ] do |t|
  chdir('peg-markdown/MarkdownTest_1.0.3') do
    sh "./MarkdownTest.pl --script=../../bin/rpeg-markdown --tidy"
  end
end

desc 'Run unit and conformance tests'
task :test => [ 'test:unit', 'test:conformance' ]


# ==========================================================
# Rubyforge
# ==========================================================

PKGNAME = "pkg/rpeg-markdown-#{VERS}"

desc 'Publish new release to rubyforge'
task :release => [ "#{PKGNAME}.gem", "#{PKGNAME}.tar.gz" ] do |t|
  sh <<-end
    rubyforge add_release wink rpeg-markdown #{VERS} #{PKGNAME}.gem &&
    rubyforge add_file    wink rpeg-markdown #{VERS} #{PKGNAME}.tar.gz
  end
end
