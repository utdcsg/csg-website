require 'html-proofer'

task :default => [:test]

task :serve do
  sh 'jekyll serve'
end

task :build do
  sh 'jekyll build'
end

task :test => [:build] do
  HTMLProofer.check_directory('./_site', {
                                :disable_external => true,
                                :url_ignore => ['#', /asset/]
                              }).run
end
