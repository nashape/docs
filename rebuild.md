
I strongly recommend a rebuild of the web application. This document discusses
the advantages and challanges to doing this.

The web application provides the following:
* Phone API
* Admin interface  
    Used internally to manage the merchant accounts and provide support to the
    consumers.
* Merchant interface  
    Used by merchants to obtain their platform marketing data.
* Consumer interface  
    Minor features for consumers such as reviewing their chop and reward
    history.

##  1. Technical Debt

The current implementation of the XXXX web application, including the phone API
endpoints, has a very high level of
[technical debt](http://martinfowler.com/bliki/TechnicalDebt.html).

Technical debt is the normal cumulation of all of the shortcuts, hacks and poor
programming practices that programmers do when building software.

The problem with technical debt is that it makes adding new features and
maintenance of the existing code much more difficult, and therefore time
consuming. The code may also become brittle – easily broken, or bugs
introduced, by small changes.

Technical debt is minimised by a process of continually improving and fixing
the code, called refactoring.

Refactoring can only be done fearlessly when there are good automated tests
covering the code.

XXXX has no automated tests, therefore refactoring is very difficult.

Because of the high level of technical debt, the lack of tests, and the
intention to add many more features to XXXX, I strongly recommend that the web
application be rebuilt from scratch using the Ruby on Rails framework (the
current web application has been built using the PHP Kohana framework).

As already decided, the phone API is to be migrated to Sinatra (a light weight
Ruby framework).

##  2. Ruby on Rails

There are many advantages (and some disadvantages) to using the Ruby on Rails
framework.

### Testing and Test Driven Development (TDD)

Rails has many excellent tools for writing automated tests and for encouraging
the TDD method.

This will ensure that tests are written as features are built, which will
ensure that the code has good test coverage to allow refactoring to be done
regularly. This in turn will ensure technical debt is kept to a minimum.

Rails also has much better support than PHP for all levels of testing: unit
testing, integration test and acceptance (user) testing.

The TDD/refactoring method produces much more robust code and the increase in
confidence allows for features to be developed faster.

### Development Speed

Developing features with Rails (including writing tests) is faster than other
frameworks because of its opinionated nature.

However the initial learning curve for Rails (and Ruby) is significant. There
are fewer good Rails developers available for hire (although the calibre of
Rails developers is usually much higher than that of PHP developers).

### Framework Structure

Rails uses a very well established MVC structure that will allow for simple
white labelling.

### Commonly Implemented Features

Rails provides opinionated solutions to common problems. These solutions have
been developed and improved over time and by many hands and can be reliably
incorporated (for example the Devise gem combined with the CanCan gem provides
a authentication and authorisation solution).

Effectively this is free outsourcing: other great developers have already
written and tested code that we can easily incoporate.

### Security

Out of the box Rails provides solutions to many typical security issues, such
as Cross Side Request Forgery (CSRF) and SQL injection.

### Database Migrations

Rails provides built in database migrations to allow easy management of
database schema changes. This includes rollbacks and tracking the complete
history of the database schema in source control.

### Asset Management

Rails provides an asset pipeline for managing assets. For production this
does asset caching and asset precompilation - merging javascript and css files
into a single file for each, and compressing these. This significantly improves
website performance.

### Community

Rails has a significantly larger active community than Kohana (PHP has
many different frameworks where Ruby has coalesced around Rails and Sinatra).

Rails is also a mature technology having been around since 2005, with Rails 4
having just been released.

This means there is a wealth of support and material to help develop in Rails.

##  3. Database

Similar to the code, the database schema has a high level of technical debt.

### Relational Schema

The database schema has grown organically. It needs a thorough redesign to
return it to a well-structured, normalised schema.

Current SQL queries have more joins than they need. This makes the queries
slower and more difficult to write.

A well-structured schema will improve the performance of all aspects of the
site (including the phone API endpoints) and make the writing of more complex
queries easier.

The efficiency gains of redesigning the database structure will allow it to
scale more effectively.

### NoSQL Database

Another option is to move to a NoSQL database such as MongoDB. A NoSQL database
allows for much more flexible schemas (which fits better with the design of the
XXXX application) and MongoDB in particular has in-built support for
geo-locations.

Replication, fail over, high availability and horizontal scaling are all easy
with MongoDB.

##  4. Agile and Goal Directed Design (GDD)

Both Agile and GDD focus the software development on meeting the needs of the
users. Doing a rebuild of the web application would allow these methods to be
followed. This will ensure only features truly required by the end user will be
built.

Another benefit will be involving all of the XXXX team in the rebuild – the
team members become both the consumers of the admin interface and the product
owners (the product owner decides the priorities for each sprint).

True user testing can be incorporated into this rebuild process because of the
iterative nature of the Agile method.

##  5. Challenges

The challanges for rebuilding include:  
* Migrating existing data.  
    The existing data will need to be migrated to the new schema.
* Time.  
    Obviously a rebuild will take some time. But fixing the existing code
    (writing tests and refactoring, as well as the database redesign) will take
    at least as long.
* Expertise.  
    We have some experience with Ruby and Rails but we will need to improve our
    expertise. This will come from simply building the site using Rails, but
    this will add time to the development process.

One option would be to evaluate Rails by re-developing a sub-section of the web
application, for example admin reports:
* Redesign the database for the relevant tables.
* Write and run the scripts to migrate the data.
* Develop with Rails.

### Migration Plan

* Database data.  
    Scripts will have to be written to convert the data to the new schema. The
    new data will need to be tested.
* API  
    The existing phone APIs will have to be maintained to allow phone apps with
    old code to still function.

    An abstraction layer will need to be written between the old code and the
    new database structure.

    Optionally the original API endpoints could be rewritten in Sinatra to talk
    directly to the new database structure.
* Testing.  
    Tests will be written as part of the rebuild but significant real world
    testing will also need to be undertaken.

    This can be done by hosting the new version of the application on a
    temporary live server and pointing development versions of the phone app to
    it.


