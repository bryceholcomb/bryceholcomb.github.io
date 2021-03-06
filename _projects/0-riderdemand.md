---
layout: post
title:  "RiderDemand"
live_url: https://rider-demand.herokuapp.com
code: https://github.com/bryceholcomb/rider_demand
stack: Ruby on Rails, Uber API, Eventful API, MapBox.js, PostgreSQL
summary: "An individual project built while a student at the Turing School of Software and Design. The application helps Uber drivers find more rides. It pulls in local event data as well as Uber ETA times to display a map pinpointing neighborhoods of high demand and low supply."
---
- Consuming multiple APIs and testing with VCR
- Caching API responses with Memcached and Dalli
- Building an API
- AJAX/jQuery to make the map and content filtering a single-page
- Proper ENV variable management with Figaro
- Continuous integration and deployment with Codeship
- PivotalTracker (albeit with an individual project)

#### Next Steps

- Wrap the current JSON endpoints in a more formal API with authentication
- Develop a mobile application that renders the RiderDemand API data
- Integrate Sidekiq and Redis to update Uber ETA times every five minutes
