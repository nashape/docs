
## Red Dot Ruby Conf 2013

These are my notes from [Red Dot Ruby Conf 2013](http://www.reddotrubyconf.com/)
focusing on the areas that I'm most interested in. I've updated much of these
notes with references.

### Slides

Here's a single page with links to many of the slides: https://gist.github.com/cheeaun/5729325

### Chef and Vagrant

"Identical Production, Staging And Development Environments Using Chef, AWS And
Vagrant" by Christopher Rigor, http://www.reddotrubyconf.com/schedule#crigor

#### Database Setup

Use a master-slave configuration where data is read from the slave and written
to the master.

#### AWS

On AWS use EBS to mount data and attach to EC2 instances. Typically store
database data on the database server or application code on the web server.

The user of a EBS allows snapshots to be created (much quicker to backup a
mysql database with an EBS snapshot compared to mysql dump) and attached to new
instances (for example for spinning up new instances to handle load spikes or
to create a staging environment for load testing).

So you would copy this EBS data onto a new staging instance then run chef to
update the settings for the staging environment.

#### Chef

Should consider pre-baking AMI images with server software, however there is
anecdotal evidence that provisioning all software on a new bare bones instance
(with chef for example) is better than upgrading the software on an existing
instance (on which upgrade conflicts can occur).

Pull db credentials out of application source control and put into chef (but
also in source control)? eg. yml files in Rails.

Should be deploying code with Chef to tell the server to pull code from github
but to do no provisioning of software.

*TODO*: What are the advantages and disadvantages of using chef_solo in
production (instead hitting a chef-server)?

#### Development

Setup the development environment to exactly match that of production – 
two VMs, one for the web server and one for the database.

*TODO*: Look into how to mount the data directory so the VM can be used by
software on the host machine, eg. using an IDE on Mac.

#### Chef 11

We should be upgrading to this as soon as possible.

#### Links

For EBS note:
http://stackoverflow.com/questions/3630506/benefits-of-ebs-vs-instance-store-and-vice-versa

See the box at http://crigor.com/reddot

See https://github.com/jedi4ever/veewee for building vagrant boxes.

See [Engine Yard Local](https://www.engineyard.com/products/local).

### APIs

See the [grape micro framework](https://github.com/intridea/grape) which
provides a DSL for developing RESTful APIs (so it's an alternative to using
Sinatra). For monitoring graph with New Relic see
[this article](http://artsy.github.io/blog/2012/11/29/measuring-performance-in-grape-apis-with-new-relic/)
and the [newrelic-graph gem](https://github.com/flyerhzm/newrelic-grape).

[This presentation](http://www.slideshare.net/dblockdotorg/building-restful-apis-w-grape)
includes discussions of modularising grape for multiple versions.

Also see http://github.com/flyerhzm/apis-benchmark ("Building Asynchronous
APIs" by Richard Huang, http://www.reddotrubyconf.com/schedule#flyerhzm,
https://speakerdeck.com/flyerhzm/building-asynchronous-apis)

For Rails, look at [jbuilder](https://github.com/rails/jbuilder) or
[RABL](https://github.com/nesquena/rabl).

### Concurrency

"Concurrency In Ruby: Tools Of The Trade" by Jose Valim,
http://www.reddotrubyconf.com/schedule#josevalim

While you do have concurrency with Ruby running on the MRI (using threads),
adding multiple cores does **not** get you parallelism. Theoretically it
should, but the GIL (Global Interpreter Lock) ensures only one thread can run
at a time.

See http://www.igvita.com/2008/11/13/concurrency-is-a-myth-in-ruby/

As it concludes in that article, the best approach is to redesign to split work
into multiple processes.

There seem to be two options,
[EventMachine](https://github.com/eventmachine/eventmachine/wiki) and
http://celluloid.io/

For performance consider asynchronous APIs - move jobs onto a queue to be
processed in the background.

See
http://www.slideshare.net/mattmatt/the-current-state-of-asynchronous-processing-with-ruby

Goliath is a non-blocking server for Ruby built on EventMachine and utilising
Ruby [Fibers](http://ruby-doc.org/core-2.0/Fiber.html). Also see the
[em-synchrony gem](://github.com/igrigorik/em-synchrony), the [em-mongo
gem](https://github.com/bcg/em-mongo) and the [sinatra-synchrony
gem](https://github.com/kyledrake/sinatra-synchrony) (which can be run with
thin).

#### Approaches to Handling Concurrency

* Use thread local variables.
* Use mutex.
* Use thread safe gems.
* Use queues to avoid shared mutable state, eg. the celluloid gem or thread.rb.

Read http://blog.paracode.com/2012/09/07/pragmatic-concurrency-with-ruby/

### Rails

#### Big Active Record Models

"Keep your ActiveRecord models manageable the Rails way" by Luismi Cavallé,
https://speakerdeck.com/cavalle/keep-your-activerecord-models-manageable-the-rails-way

Reduce the size of your Active Record models.

* Use a service layer - [DCI](http://dci-in-ruby.info/),
  [Hexagonal](http://blog.mattwynne.net/2012/05/31/hexagonal-rails-objects-values-and-hexagons/).
* [ActiveSupport Concerns](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)
  and read [DHH's article](http://37signals.com/svn/posts/3372-put-chubby-models-on-a-diet-with-concerns).
  Concerns are the "official" solution to fat models in Rails 4. [Read an argument](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
  against concerns.
* Workflow and Scheduling.
* [Service objects](http://stevelorek.com/service-objects.html) to avoid
  callbacks.
* Form objects that extend Active Model and wrap more than one Active Record
  model (need to put validation in Form object).

### Javascript

Client side MVCs such as angular.js and ember.js. There's also
sencha ext js (and the netzke component for it) but you have to pay for it.

#### Gems

* The [gon gem](https://github.com/gazay/gon) to more easily get Rails data to
  the front end javascript.
* The [dalli gem](https://github.com/mperham/dalli) for memcached.

#### Security

Instead of using `secret_token.rb`, put token into `ENV['SECRET_TOKEN']`.

### Testing

#### View Testing

View testing is aimed at checking authentication and authorisation. It's not
particularly easy with current tools, such as view specs and render_views for
RSpec. The [fracture gem](https://github.com/nigelr/fracture) is designed to
unify view testing.

#### Gems

* The [cookpad/kage gem](https://github.com/cookpad/kage) which provides HTTP
  shadow requests for Blue-Green deployment (and testing). There's also a 
  [puppet-kage gem](https://github.com/hsbt/puppet-kage) for it.
* The [glint gem](https://github.com/kentaro/glint) from Kentaro Kuribayashi,
  https://speakerdeck.com/kentaro/glint (it also works with RSpec).

Also handy: `RSpec::Given::use_natural_assertions`.

### Ruby

#### Ruby 2.0

"Ruby 2.0 on Rails in Production" by Akira Matsuda,
https://speakerdeck.com/a_matsuda/ruby-2-dot-0-on-rails-in-production

No reason not to put it in production (it's at p195 now).

* The default encoding in Ruby 2.0 is utf-8.
* Now has [keyword args (hash)](https://briandamaged.org/blog/?p=1243) aka
  kwargs.
* It has a debug_inspector API. The debugger gem (which works with Rails)
  [doesn't work with Ruby 2.0](https://github.com/cldwalker/debugger/issues/47#issuecomment-14965459)
  and the [debugger2 gem](https://github.com/ko1/debugger2) doesn't work
  properly with Rails yet. Although the [byebug gem](https://github.com/deivid-rodriguez/byebug)
  seems to work with both.
* Supposedly the [redmine4](https://github.com/asakusarb/redmine4ruby-lang)
  fork of redmine runs on Ruby 2.0.

#### Code

Try:  
`p str: s, size: s.size`

#### Miscellanenous

* [Semantic Versioning](http://semver.org/).
* The [rbenv gemset plugin](https://github.com/jamis/rbenv-gemset).

