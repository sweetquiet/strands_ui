# STRANDS User Interfaces

User interfaces for the robots, displayed using a web browser.

# Requirements

`rosdep` should give you most requirements, but we don't yet have a rosdep for [web.py](http://webpy.org) which is required by `strands_webserver`. This can be installed using, e.g., `sudo pip install web.py`


# `strands_webserver`

The `strands_webserver` is a node which both acts as a webserver and a ROS node which can receive input from other nodes. This allows these other nodes to control what the webserver displays and receive feedback from user interaction with the pages.

## Running

To run the server, first run rosbrige if you don't have it running already.
```
roslaunch rosbridge_server rosbridge_websocket.launch
```
Then run the server
```bash
rosrun strands_webserver strands_webserver 
```

This will launch the webserver on localhost port 8090, so browse to [http://localhost:8090](http://localhost:8090) and you should see something like
```text
Ready; display = 1
```

## Displaying content

The webserver manages individual *displays* separately, one for each request to its URL. These can be published to individual by specifying their number, or you can just send 0 to publish to all of them.

Interaction with `strands_webserver` typically happens by publishing instructions to it for display. These can either be URLs or bits of html which will be injected into its main page. The URLs are loaded in a full size iframe. The bits of html are injected as the `innerHTML` of the `content` node in the main page (`strands_webserver/data/main.html`).

The example below shows how to publish a simple html page to the webserver, both a normal URL (given that it is happy to appear in an iframe) then a page defined locally.
```python
#! /usr/bin/env python

import sys
import os
import roslib
import rospy

import strands_webserver.client_utils

if __name__ == '__main__':
	rospy.init_node("strands_webserver_demo")
	# The display to publish on, defaulting to all displays
	display_no = rospy.get_param("~display", 0)	

	# display a start-up page
	strands_webserver.client_utils.display_url(display_no, 'http://strands-project.eu')

	# sleep for 2 seconds
	rospy.sleep(5.)

	# tell the webserver where it should look for web files to serve
	http_root = os.path.join(roslib.packages.get_pkg_dir("strands_webserver"), "data")
	strands_webserver.client_utils.set_http_root(http_root)

	# start with a basic page pretending things are going normally
	strands_webserver.client_utils.display_relative_page(display_no, 'example-page.html')
```

To display arbitrary HTML you can use calls like the following
```python
strands_webserver.client_utils.display_content(display_no, '<p>Hello world</p>') 
```

