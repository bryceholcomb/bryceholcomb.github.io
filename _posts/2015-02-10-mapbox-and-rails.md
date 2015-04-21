---
layout: post
title:  "MapBox.js and Rails: Part One"
date:   2015-02-10 21:35:07
summary: "Moving data from your database to a map using AJAX"
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
# app/controllers/events_controller.rb

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
# app/controllers/events_controller.rb

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
of formats that are specified in the request. This endpoint still allows us to
handle HTML requests and default to the normal behavior of finding the
corresponding view and sending it as HTML if you still have something like an
Events index page.

Next we need to transform our @events from an
[ActiveRecord::Relation](http://api.rubyonrails.org/classes/ActiveRecord/Relation.html) into
GeoJSON objects to be passed to the template to be rendered on the map as
markers.

{% highlight ruby %}
# app/controllers/events_controller.rb
events_controller.rb

class EventsController < ApplicationController
  def index
    @events = Event.all
    @geojson = Array.new
    build_geojson(event, @geojson)
  end

  respond_to do |format|
    format.html
    format.json { render json: @geojson }
  end

  def build_geojson(events, geojson)
    events.each do |event|
      geojson << GeojsonBuilder.build_event(event)
    end
  end
end

# app/models/geojson_builder.rb
class GeojsonBuilder
  def build_event(event, geojson)
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

This next bit of code iterates over each event in the collection and the
GeojsonBuilder creates a JSON object and adds it to the geojson array. In my
project I never saved the events to my database. I merely just consumed Eventful's
API and manipulated the data within my server to fit my needs. For this example
I am mimicking what it would look like to serve events from a database so I
don't have to go into the dirty details on consuming external APIs. All this being
said, if you already have data or intend to save your data to the database, one simpler approach might be a custom serializer that converts your records into GeoJSON rather than
rolling your own "Geojson Builder".

Now that we have GeoJSON that we can pass to our template, we can move on to the
JavaScript that will load the events onto the page and turn them into points on
the map.

### Adding the Event data to the DOM

{% highlight javascript %}
# app/assets/javascripts/map.js
$(document).on("ready", function() {
  L.mapbox.accessToken = 'your access token';
  var map = L.mapbox.map('map', 'Your map layer', { zoomControl: false })
  .setView([39.739, -104.990], 12);

  map.featureLayer.on("ready", function(e) {
    getEvents(map);
  });
});

function getEvents(map) {
  var $loading_wheel = $("#spinning-wheel")
  $loading_wheel.show();
  $.ajax({
    dataType: 'text',
    url: '/events.json',
    success:function(events) {
      $loading_wheel.hide();
      var geojson = $.parseJSON(events);
      map.featureLayer.setGeoJSON({
        type: "FeatureCollection",
        features: geojson
      });
      addEventPopups(map);
    },
    error:function() {
      $loading_wheel.hide();
      alert("Could not load the events");
    }
  });
}

function addEventPopups(map) {
  map.featureLayer.on("layeradd", function(e){
    var marker = e.layer;
    var properties = marker.feature.properties;
    var popupContent = '<div class="marker-popup">' + '<h3>' + properties.title + '</h3>' +
                       '<h4>' + properties.address + '</h4>' + '</div>';
    marker.bindPopup(popupContent, {closeButton: false, minWidth: 300});
  });
}
{% endhighlight %}

The first third of this JavaScript file is more of the maestro that orchestrates
the functions I have defined below. First it waits for the DOM to render which is the
jQuery "$(document).on('ready')" function that takes a callback including all
the behavior you want to occur after the DOM loads. Within this callback, I
initialize the map and then once the map is loaded, call the getEvents function
which begins the process of adding events to the map.

For kicks, I added a loading wheel gif that I display before the AJAX call and
then hide it again after the call was either successful or failed. The AJAX call
itself takes quite a few parameters, but all we need is the datatype and url.
The success and error portions are just callbacks that will be run depending on
if the call was...you guessed it, successful or failed. Within the success
callback, I pass in the geoJSON that is returned as "events". That way I can
call methods on that collection of objects and add them to the feature layer.
After adding them to the map as markers, I call my addEventPopups method that
generates markup and interpolates the marker properties I defined within the
markup.

This concludes the first part of this two part tutorial. It is pretty simple as
MapBox does a lot of the heavy lifting for us. I highly recommend checking out
the [docs](https://www.mapbox.com/mapbox.js/api/v2.1.8/) because there is quite
a bit more functionality and customization that I haven't covered here. In the
next part of the tutorial I will cover how I made the map page into more of a
single page application with jQuery listeners that filter the events and show
how I geocoded neighborhoods onto the map and dynamically color them based on
ETA times received from Uber's API. Best of luck with your mapping and feel free
to [contact](http://bryceholcomb.com/contact) me with any questions.
