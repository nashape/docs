
A complete rebuild of the web application is strongly recommended. This
document discusses the advantages and challanges to this.

##  1. Technical Debt

The current implementation of the XXXX web application, including the phone API
endpoints, has a very high level of technical debt.

Technical debt is the cumulation of all of the shortcuts, hacks and poor
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
application, including the phone API endpoints, be rebuilt from scratch using
the Ruby on Rails framework (the current web application has been built using
the PHP Kohana framework).

##  2. Ruby on Rails

There are many advantages (and some disadvantages) to using the Ruby on Rails
framework.

### Testing and Test Driven Development (TDD)

Rails has many excellent tools for writing automated tests and for enforcing
the TDD method.

This will ensure that tests are written as features are built, which will
ensure that the code has good test coverage to allow refactoring to be done
regularly. This in turn will ensure technical debt is kept to a minimum.

Rails also has much better support than PHP for all levels of testing: unit
testing, integration test and acceptance (user) testing.

### Community

Rails has a significantly larger active community than Kohana (PHP has
many different frameworks where Ruby has coalesced around Rails and Sinatra).

Rails is also a mature technology having been around since 2005, with Rails 4
being released in the near future.

This means there is a wealth of support and material to help develop in Rails.

### Commonly Implemented Features

Rails provides opinionated solutions to common problems. These solutions have
been developed and improved over time and by many hands and can be reliably
incorporated (for example the Devise gem which provides a complete
authentication and authorisation solution).

### Development Speed

Because of its opinionated nature, developing features with Rails (including
writing tests) is faster than other frameworks.

However the initial learning curve for Rails (and Ruby) is significant. There
are many fewer good Rails developers available for hire (although the calibre
of Rails developers is usually much higher than that of PHP developers).

### Security

Out of the box Rails provides solutions to many typical security issues, such
as Cross Side Request Forgery (CSRF) and SQL injection.

### Database Migrations

Rails provides built in database migrations to allow easy management of
database schema changes.

##  3. Database

Similar to the code, the database schema has a high level of technical debt.

### Relational Schema

The database schema has grown organically. It needs a thorough redesign to
return it to a well-structured schema.

Current SQL queries have more joins than they need. This makes the queries
slower and more difficult to write.

A well-structured schema will improve the performance of all aspects of the
site (including the phone API endpoints) and make the writing of more complex
queries easier.

### NoSQL Database

Another option is to move to a NoSQL database such as MongoDB. A NoSQL database
allows for much more flexible schemas (which fits better with the design of the
XXXX application) and MongoDB in particular has in-built support for
geo-locations.

##  4. Challenges

The challanges for rebuilding include:  
* Time.
* Migrating existing data.
* Expertise.

### Migration Plan

* database data
* features/rollout
* testing

