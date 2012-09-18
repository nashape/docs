
# Rails Install HOWTO

This document describes how to install RVM, Rails and Guard/Growl on a linux
box, such as a Ubuntu virtual machine running on Mac OSX.

I'm little better than a novice at this, so if you use this document, YMMV.

## Setting up RVM gemset for Rails

1.  Ruby  
To show current installations:

    ```
    rvm list
    rvm install 1.9.3
    rvm use 1.9.3 --default
    ```
I'm currently running 1.9.3

2.  Gemsets  
Create a gemset for rails:

    ```
    rvm gemset create rails32
    ```
Create a gemset for sinatra:

    ```
    rvm gemset create sinatra
    ```
List gemsets:

    ```
    rvm gemset list
    ```
Note, we will use one gemset per project. See step 3. of "Setting up a new
Rails App" below for creating a project based .rvmrc file.

3.  Installing Rails  
Ensure you're using the correct gemset:

    ```
    rvm use ruby-1.9.3-p194@rails32
    ```
Before installing:

    ```
    gem install rails
    ```

4.  RVM Wrapper for unicorn  
We need an RVM wrapper for unicorn (for development, I run unicorn in a
separate screen with the output directed to STDOUT) so that the correct version
of Ruby and gems is loaded when unicorn runs.  
Create the wrapper with the correct gemset:

    ```
    rvm wrapper ruby-1.9.3-p194@rails32 r193 unicorn
    ```

## Setting up a new Rails App

1.  Create rails app

    ```
    rails new newapp -O
    ```
-O = skip active record since we will be using mongo
And change directory to the app root dir:

    ```
    cd newapp
    ```

2.  Create project rvmrc

    ```
    rvm --create --rvmrc 1.9.3@rails32
    ```
Use the correct version of ruby and the rails gemset. You will need to leave
the dir (eg. cd ..) then return and answer 'yes' to use the rvmrc file.

4.  Create a new github repo

    ```
    git init
    git add .
    git commit -m 'initial checkin'
    git remote add origin git@github.com:<user>/newapp.git
    git push -u origin master
    ```

5.  Edit Gemfile  
After this line:

    ```
    # gem 'rails', :git => 'git://github.com/rails/rails.git'
    ```
Add:

    ```
    gem 'mongo'
    ```
Uncomment:

    ```
    gem 'therubyracer', :platforms => :ruby
    ```
Uncomment:

    ```
    gem 'unicorn'
    ```
At the end add:

    ```
    gem 'mongoid'
    gem 'bson_ext'

    # Acceptance testing with cuke
    group :test do
      gem 'cucumber-rails', :require => false
      gem 'database_cleaner'
      gem 'factory_girl_rails'
      gem 'simplecov'
      gem 'single_test'
    end
    ```

6.  Run bundle

    ```
    bundle install
    ```

7.  Create the file config/unicorn.rb with:

    ```
    APP_ROOT = File.expand_path(File.dirname(File.dirname(__FILE__)))

    if ENV['MY_RUBY_HOME'] && ENV['MY_RUBY_HOME'].include?('rvm')
      begin
        rvm_path = File.dirname(File.dirname(ENV['MY_RUBY_HOME']))
        rvm_lib_path = File.join(rvm_path, 'lib')
        # $LOAD_PATH.unshift rvm_lib_path
        require 'rvm'
        RVM.use_from_path! APP_ROOT
      rescue LoadError
        raise "RVM ruby lib is currently unavailable."
      end
    end

    ENV['BUNDLE_GEMFILE'] = File.expand_path('../Gemfile', File.dirname(__FILE__))
    require 'bundler/setup'

    worker_processes 4
    working_directory APP_ROOT

    preload_app true

    timeout 30

    # listen APP_ROOT + "/tmp/sockets/unicorn.sock", :backlog => 64
    listen 'unix:/tmp/unicorn_newapp.sock', :backlog => 512

    pid APP_ROOT + "/tmp/pids/unicorn.pid"

    # stderr_path APP_ROOT + "/log/unicorn.stderr.log"
    # stdout_path APP_ROOT + "/log/unicorn.stdout.log"

    before_fork do |server, worker|
      defined?(ActiveRecord::Base) && ActiveRecord::Base.connection.disconnect!

      old_pid = APP_ROOT + '/tmp/pids/unicorn.pid.oldbin'
      if File.exists?(old_pid) && server.pid != old_pid
        begin
          Process.kill("QUIT", File.read(old_pid).to_i)
        rescue Errno::ENOENT, Errno::ESRCH
          puts "Old master alerady dead"
        end
      end
    end

    after_fork do |server, worker|
      defined?(ActiveRecord::Base) && ActiveRecord::Base.establish_connection
    end
    ```

8.  Create mongoid config

    ```
    rails g mongoid:config
    ```
This creates the config/mongoid.yml file which may need to be edited.

9.  Create cucumber files

    ```
    rails g cucumber:install --skip-database
    ```
Installs the features directory for cucumber.

10. Add the file features/support/database_cleaner.rb for cucumber, with:

    ```
    require 'database_cleaner'
    DatabaseCleaner.strategy = :truncation
    DatabaseCleaner.orm = "mongoid"
    Before { DatabaseCleaner.clean }
    ```

11. Update test/test_helper.rb and add to the start of the file:

    ```
    require 'database_cleaner'
    ```
Then at the end of the file:
    ```
    class ActiveSupport::TestCase
      # Add more helper methods to be used by all tests here...
      def setup
        DatabaseCleaner.start
      end

      def teardown
        DatabaseCleaner.clean
      end
    end
    ```

12. Edit features/support/env.rb  
Comment out the lines:

    ```
    # begin
    #   DatabaseCleaner.strategy = :transaction
    # rescue NameError
    #   raise "You need to add database_cleaner to your Gemfile (in the :test
    #   group) if you wish to use it."
    # end
    ```

13. Update the .gitignore so it has:

    ```
    *.rbc
    *.sassc
    .sass-cache
    capybara-*.html
    .rspec
    /.bundle
    /vendor/bundle
    /log/*
    /tmp/*
    /db/*.sqlite3
    /public/system/*
    /coverage/
    /public/coverage/
    /spec/tmp/*
    **.orig
    rerun.txt
    pickle-email-*.html
    ```

14. For simplecov, add a .simplecov file to the app directory:

    ```
    SimpleCov.start 'rails' if ENV["COVERAGE"]
    SimpleCov.coverage_dir 'public/coverage'
    ```
Edit features/support/env.rb and at the start of the file add:

    ```
    require 'simplecov'
    ```
Edit test/test_helper.rb and at the start of the file add:

    ```
    require 'simplecov'
    ```
Note: coverage will only be generated with the command:

    ```
    COVERAGE=true rake test
    ```
Also edit the .gitignore file and add (if it is not already there):

    ```
    /coverage/
    /public/coverage/
    ```

15. Use scaffold to generate a model:

    ```
    rails g scaffold user email:string name:string admin:boolean salt:string password_hash:string -r factory_girl
    ```
This will also create a user factory in test/factories.

16. Factory Girl  
The test/functional/users_controller_test.rb tests will fail. Edit that file,
replace this:

    ```
    setup do
      @user = users(:one)
    end
    ```
With:

    ```
    setup do
      # writes to the test db
      @user = FactoryGirl.create(:user)
    end
    ```
And obviously do this wherever you use FactoryGirl instead of a fixture.

## Guard and Growl

1.  Add gems to Gemfile

    ```
    group :tools do
      gem 'guard'
      gem 'rb-inotify', '>= 0.8.8' # don't know what this is but got an error without it
      gem 'guard-cucumber' # for listening to cucumber tests
      gem 'guard-test' # for listening to test::unit tests
      gem 'ruby_gntp' # notifier for growl
    end
    ```

2.  Add cucumber settings to the guard file

    ```
    guard init cucumber
    ```
Since there is no Guardfile, this will create one.

3.  Add test::unit settings to the guard file

    ```
    guard init test
    ```

4.  Add ruby-prof  
Add this gem to the ':test' group in the Gemfile:

    ```
    gem 'ruby-prof'
    ```
Without this you get an error when guard runs the unit tests:

    ```
    Specify ruby-prof as application's dependency in Gemfile to run benchmarks.
    ```

5.  Setup growl to receive the notifcations  
I'm using growl 1.4 (yes, I paid a few dollars for it, but this is a very
useful application and the devs deserve money for it).

    ```
    Open Growl Preferences
    Select the 'Network' tab
    Check 'Listen for incoming notifications'
    Provide a password (actually you can leave this blank)
    ```
The available IP addresses, one of which you need to supply in the
configuration (see below), are listed on the right.  

6.  Configure Guardfile for notifcation

    ```
    notification :gntp, :sticky => true, :host => '192.168.1.1', :password => 'password'
    ```
Or if you don't use a password:
    ```
    notification :gntp, :sticky => true, :host => '192.168.1.1', :password => ''
    ```
The IP address is show in the Growl Preferences panel (on Mac).

7.  Configure 'features/support/env.rb' for 'Test::Unit automatic runner' error  
When running cucumber under guard (which executes it as ```bundle exec
cucumber```), this occurs:

    ```
    Running all features
    Using the default profile...
    0 scenarios
    0 steps
    0m0.000s
    invalid option: --profile
    Test::Unit automatic runner.
    ...
    ```
To fix this, add this to the end of 'features/support/env.rb':

    ```
    Test::Unit.run = true
    ```
Update: actually this is an issue when there are all passing tests, including
if there are no tests at all. This makes sense - it's getting to the end of the
tests then trying to run the unit tests. But if you include the line above, you
can no longer run cucumber from the command line.  
So my (very hacky) solution is now:  

    ```
    if !ENV['GUARD_NOTIFY'].nil? && ENV['GUARD_NOTIFY'] == 'true'
      Test::Unit.run = true
    end
    ```
Further update: I've noticed that if I run guard with the notifications off,
there is no problem. When the notifications are on, I still get the same error.

8.  Run guard  
One option is to simply run guard from the command line:

    ```
    guard -c -B
    ```
Without the ```-B``` flag it will show a message about running under 'bundle
exec':

    ```
    bundle exec guard -c
    ```

