# Docker Project setup

## Not converted for use with moOde.

Running MoOde, or moOde processes in a docker container may allow for quicker
testing, but is not part of a current (or planned) workflow.

This document exists to capture any experimental efforts that may be taken.


- Have your local `.env` file point to the Docker database host:
  `DB_HOST=db`
- To boost performance between container and host, add `docker-sync` as a dependency in the project's `Gemfile`:
  `gem 'docker-sync', '~> 0.5.7'`
- Install the dependency:
  `$ bundle install`
  (if you don't have Bundler, run `$ gem install bundler` first)



Make sure you have the following files in your project:

- `docker-compose.yml`
- `docker-compose-dev.yml`
- `docker-sync.yml`

@todo: we will author a generic scaffold for these files.
For now, you can copy them from a previous recent project.
