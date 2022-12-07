# docker-rails-demo
Just a dev project to learn more about docker and rails.

Boiler plate project from (here)[https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application].

## Goals

My long term vision for this project involves accomplishing a number of goals:
- explore docker independently by creating a dockerized application (ui, database, server, cache, etc.).
- build a chat application whose users and messages are modeled in a Cassandra database.
- create another container that can test the performance of the system under load by, for example, firing messages rapidly into the system, or creating many concurrent connections. Or both!
- to have this container run its own tests and reports. This could be done to compare how performant the system is when using a particular database compared to a different one.
- Have a (relatively) nice UI from which to set up and run these tests and to view reports.

## How to run

Haven't tested these steps yet, but this is what I would do:

`docker compose up --build`

`docker­ compose run drkiq rake db:reset`

`docker­ compose run drkiq rake db:migrate`

`docker compose up`


## How it went and what I got done

For a decent chunk of the first day working on this project, I was trying to figure out why not all the containers would successfully start when 
running `docker-compose up`. 

The logs showed that the sidekiq container would run into an error while starting the sidekiq gem. The error read:
`heartbeat: Unsupported command argument type: FalseClass`.

This appeared to be a known issue with the version of the gem as it was, 6.4.2. Articles recommended upgrading to 
6.5.5.

I tried changing the version in the Gemfile to 6.5.5 and restarting the containers. It was my understanding that
bundle install is run when containers are started.

When I used `docker exec -it [sidekiq container id] bash` to investigate, the Gemfile in the container showed the
Gemfile to have sidekiq version 6.4.2 still, although in my local filesystem the version number was clearly 6.5.5.

This stumped me. Knowing the interview was coming soon, I had to try something different.
In the Dockerfile.rails file, I added an extra line:
`RUN gem install sidekiq -v 6.5.5`,
then I recreated all the containers.

I am currently still not sure whether:
- this changed line was read by Docker while recreating the containers, thus installing the correct version
- docker can only install the specified gems with their versions when the containers are initially created

either way, this was quite strange to me and I would love to experiment more to get a better understanding of
what happened here. I would repeat similar steps to above, perhaps without that added line, to see what changed.

Seems like a bad limitation to not be able to change version numbers of gems without destroying and recreating
the containers. I'm sure there's a better way, and I would love to keep experimenting with docker to find out the
best way to do this.

Anyways, now it was working. This means that now I could create a new independent container for the UI that
could talk to the rails server. 