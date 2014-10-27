---
layout: post
title: "A Quick Path from CSV to YAML"
categories: csv yaml
---

It has been several times now that I've had to convert a CSV file into YAML. Even though the task is quite simple, I always forget the details and end up reading through the [CSV documentation](http://ruby-doc.org/stdlib-1.9.3/libdoc/csv/rdoc/CSV.html), which has a ton of good information, but lacks a short explanation of what I want. So I spin my wheels for a bit and then finally figure out some ugly algorithm that produces a YAML file. Now that it's been several times, though, it's time to figure out how to do it the right way.

To start, let's assume we have a CSV file of weather data called `weather_data.csv` which we want to convert to a YAML file. The first fews lines look like the following:

```
date,mean_temp,max_temp,min_temp,dew_point,precipitation,wind_speed,max_wind_speed,visibility,sun_rise,sun_set,day_length
2012-01-01,8,10,6,7,0.6,20,30,11.5,8:13 AM GMT,3:57 PM GMT,7h 44m
2012-01-02,4,6,3,2,0.0,21,35,11.5,8:13 AM GMT,3:58 PM GMT,7h 45m
```

To handle this data, we need the `CSV` and `YAML` classes. We'll pull those in first:

``` ruby
require 'csv'
require 'yaml'
```

To read in the weather data, we simply use the `CSV::read` method. A key detail is in the options hash which we pass to the read method:

``` ruby
data = CSV.read('weather_data.csv', :headers => true)
```

By turning on the headers, `CSV::read` will return an array of `CSV::Row` objects, each with the appropriate header connected to the appropriate data within a line. From here, it's simply a matter of converting each of the `CSV::Row` objects into hashes, a conversion we'll add to the end of our most recent line.

``` ruby
data = CSV.read('weather_data.csv', :headers => true).map(&:to_hash)
```

Now that we have all our data loaded into memory as an array of hashes, it's a short way to the YAML file we want.

``` ruby
File.open('weather_data.yml', 'w') { |f| f.write(data.to_yaml) }
```

The key here is the `to_yaml` method which becomes available with the inclusion of the `YAML` class. Finally, we wrap up the conversion within a block passed to the `File::open` method. Note that by handling our IO within a block, we don't have to remember to close our handle on the file. The end of the block does that for us.

The resulting script all together is exceedingly simple and makes the conversion a breeze. No more wheel-spinning. In full, the script is as follows:

``` ruby
require 'csv'
require 'yaml'

data = CSV.read('weather_data.csv', :headers => true).map(&:to_hash)

File.open('weather_data.yml', 'w') { |f| f.write(data.to_yaml) }
```
