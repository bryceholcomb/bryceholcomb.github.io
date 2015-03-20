---
layout: post
title:  "Making MapBox.js Play Well with Rails"
date:   2015-02-10 21:35:07
summary: "Populating a map with data from your database using AJAX"
---
Last week, I began work on an individual project for Turing,
[RiderDemand](https://www.rider-demand.herokuapp.com). Check out my
[Projects](https://www.bryceholcomb.com/projects/) page for more info. I planned
to pull event data from an external API into my application and plot the events on a map. I have used Google
Maps on a previous project, so I figured I would give Leaflet.js and MapBox.js a try. (MapBox is just
a handy layer wrapped around Leaflet's open source JavaScript library).

I unfortunately, or fortunately depending on how you look at it, couldn't find too many resources on integrating MapBox with a Ruby on Rails
application, so I figured I should add to the mix. The code samples are pulled
from RiderDemand and follow the process of pulling events from the database
asynchronously
and rendering them on the map.

This "tutorial" assumes that you have a few things set up:

- You have created a MapBox account with an api key
- The data you are using have float type latitude and longitude attributes

### Creating a JSON endpoint and GeoJSON objects

{% highlight ruby %}
events_controller.rb

class EventsController < ApplicationController
  def index
    @events = Event.all
  end
end
{% endhighlight %}

The above code is your typical RESTful endpoint for a Rails application. With no format specified,
Rails knows to respond with HTML and render a template named "index.html.erb"
from the "views/events" folder. We simply want this endpoint to respond with
JSON data, more specifically GeoJSON data as per the MapBox docs, though when we make an [AJAX](http://api.jquery.com/jquery.ajax/) call to it.

{% highlight ruby %}
events_controller.rb

class EventsController < ApplicationController
  def index
    @events = Event.all

    respond_to do |format|
      format.html
      format.json { render json: # some JSON data }
    end
  end
end
{% endhighlight %}

First we need to tell this endpoint how to respond to a JSON response. The
method #respond_to is built in with Rails and will handle the different types
of formats that are specified in the request.

Next we need to transform our @events from an
[ActiveRecord::Relation](http://api.rubyonrails.org/classes/ActiveRecord/Relation.html) into
GeoJSON objects to be passed back to the template to be rendered on the map.

{% highlight ruby %}
events_controller.rb

class EventsController < ApplicationController
  def index
    @events = Event.all
    @geojson = Array.new
    build_geojson(@events, @geojson)

  def build_geojson(events, geojson)
    geojson << {
      type: "Feature",
      geometry: {
        type: "Point",
        coordinates: [event.longitude, event.latitude] # this part is tricky
      },
      properties: {
        title: event.title,
        address: event.address,
        :"marker-color" => "#FFFFFF",
        :"marker-symbol" => "circle",
        :"marker-size" => "medium",
      }
    }
  end
end
{% endhighlight %}


