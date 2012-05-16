release_ops_path = File.expand_path('../releaseops/lib', __FILE__)

# if the special submodule is availabe, use it
# we use a submodule because it doesn't depend on anything else (*cough* bundler)
# and can be shared across projects
#
if File.exists?(release_ops_path)
  require File.join(release_ops_path, 'releaseops')

  # sets up the multi-ruby zk:test_all rake tasks
  ReleaseOps::TestTasks.define_for(*%w[1.8.7 1.9.2 jruby rbx ree 1.9.3])

  # sets up the task :default => 'spec:run' and defines a simple
  # "run the specs with the current rvm profile" task
  ReleaseOps::TestTasks.define_simple_default_for_travis

  # Define a task to run code coverage tests
  ReleaseOps::TestTasks.define_simplecov_tasks

  # set up yard:server, yard:gems, and yard:clean tasks 
  # for doing documentation stuff
  ReleaseOps::YardTasks.define


  namespace :zk do
    namespace :gems do
      task :build do
        require 'tmpdir'

        raise "You must specify a TAG" unless ENV['TAG']

        ReleaseOps.with_tmpdir(:prefix => 'zk') do |tmpdir|
          tag = ENV['TAG']

          sh "git clone . #{tmpdir}"

          orig_dir = Dir.getwd

          cd tmpdir do
            sh "git co #{tag} && git reset --hard && git clean -fdx"

            sh "rvm 1.8.7 do gem build zk.gemspec"

            mv FileList['*.gem'], orig_dir
          end
        end
      end

      task :push do
        gems = FileList['*.gem']
        raise "No gemfiles to push!" if gems.empty?

        gems.each do |gem|
          sh "gem push #{gem}"
        end
      end

      task :clean do
        rm_rf FileList['*.gem']
      end

      task :all => [:build, :push, :clean]
    end
  end


  task :clean => 'yard:clean'
end

# cargo culted from http://blog.flavorjon.es/2009/06/easily-valgrind-gdb-your-ruby-c.html
VALGRIND_BASIC_OPTS = %w[
  --num-callers=50
  --error-limit=no
  --partial-loads-ok=yes
  --undef-value-errors=no
  --trace-children=yes
  --track-fds=yes
  --tool=memcheck
].join(' ')
  #--leak-check=yes

task :valgrind do
  require 'time'
  require 'date'

#   logfile = File.join('/tmp', Time.now.strftime('%Y%m%d_%H%M%S_zk_valgrind.out'))

  logfile = '/tmp/valgrind.out'

  File.open('/tmp/valgrind.out', 'w') do |fp|
    sh "valgrind #{VALGRIND_BASIC_OPTS} #{ENV['VALGRIND_EXTRA_OPTS']} --log-fd=#{fp.to_i} rspec spec"
  end
end

