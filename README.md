# appdaemon_pixelt
There is a marvelous project [PixelIT](https://www.bastelbunker.de/pixel-it/) which is a DIY 8x32bit RGB WLAN LED matrix to display text and graphics for around 50€. The screen is more or less dumb, you have to feed it with data by a server.
The original developer provides a node-red flow to do so. The hardware and firmware is original from this website.
The wiki is in german, to download the firmware use this link [ESP8266-Firmware](https://www.bastelbunker.de/wp-content/uploads/PixelIt.zip)

## Homeassistant
I did not manage to feed data via node-red to the display so i started to develop an appdaemon class for this task. The result is in an early stage but is working. See a demo on [Youtube](https://youtu.be/CrZR8chQrP4).

### Concept

The appdaemon shows a "screen" after another. A screen is assembled from a json [template](https://wiki.dietru.de/books/pixel-it/page/apiscreen) to set static information like colors and a "dynamic" part the message. E.g. you can design a template for temperature with a special icon or animation, text color and display duration (lifetime) and feed the actual temperature value via home-assistant. Screennames should be unique if you update data dynamicaly.

### Setup
You need a running appdaemon, see e.g. [tutorial](https://webworxshop.com/getting-started-with-appdaemon-for-home-assistant/)
Install the class to  your appdaemon an configure it:

```yaml
pixelit:
  module: pixelit
  class: pixelit
  ip: 192.168.178.24
  path: "/config/pixelit/"
  entitiy_id: sensor.pixelit
  debug: False
```
parameter | meaning
----------|----------
ip|ip of the pixel controller
path| the path to the json templates, this has to be outside of the appdaemon/apps/pixelit folder!!
entity_id| entity in HA to update with information about the display

#### Example Homeassistant config:
configuration.yaml
```
sensor:
- platform: rest
  resource: http://192.168.178.47:5050/api/appdaemon/pixelit_sensor
  method: POST
  headers:
    Content-Type: "application/json"
  name: Pixelit
  value_template: > 
      {{ value_json.state }}
  scan_interval: 300

notify:
  - name: pixelit
    platform: rest
    method: POST_JSON
    headers:
      Content-Type: "application/json"
    resource: http://192.168.178.47:5050/api/appdaemon/pixelit_add
    title_param_name: "title"

rest_command:
  pixelit_delete:
    url: http://192.168.178.47:5050/api/appdaemon/pixelit_delete
    method: POST
    payload: '{"title": "{{ title }}"}'
    content_type: 'application/json; charset=utf-8'

  pixelit_update:
    url: http://192.168.178.47:5050/api/appdaemon/pixelit_update
    method: POST
    payload: '{"title": "{{ title }}","message": "{{ message }}"}'
    content_type: 'application/json; charset=utf-8'

  pixelit_next:
    url: http://192.168.178.47:5050/api/appdaemon/pixelit_next
    method: POST
    payload: '{}'
    content_type: 'application/json; charset=utf-8'

switch:
  - platform: rest
    name: pixelit_sleepmode
    resource: http://192.168.178.47:5050/api/appdaemon/pixelit_sleepmode
    body_on: '{"sleepMode": "true"}'
    body_off: '{"sleepMode": "false"}'
    is_on_template: '{{ value_json.sleepMode == True }}'
    headers:
      Content-Type: application/json
```
**use your own homeassistant ip instead of 192...47!!!**

### Usage

Create the templates according to the [wiki](https://wiki.dietru.de/books/pixel-it/page/apiscreen) of pixelit and add these two values:

```json
    "repeat": 10,
    "lifetime": 15,
 ```
parameter | meaning
----------|----------
repeat|how often will this screen be shown
lifetime|how long is this screen visible

Name the json-files according to your screen an put it in the defined path.

#### Templates
There are some samples in the templates subdirectory. Use them as startingpoint for your own screens. You can test them in the browser in the test-area of your pixelit. 

**Hint** Don't use `position` in the text-section, but set `"scrollText": "auto"` so text will be scrolled if to long to display at once.

**Important** Take care of the syntax, the screens must be valid JSON!!

#### To add a screen use the service:

*service:* notify.pixelit
with e.g.
```
title: "alarm"
message: "Door is open!"
```
parameter | meaning
----------|----------
title|selects the "*title*.json" template and uses it as the screen name
message|the text to display
target| (optional) "warning" and "alert" emphasises the message by repeating it more often

#### To delete a screen from the playlist

*service:* rest_command.pixelit_delete

parameter | meaning
----------|----------
title|selects screen to delete first match is deleted, so unique screen names are useful

#### To update a screen message text

*service:* rest_command.pixelit_update

parameter | meaning
----------|----------
title|selects screen to update first match is used, so unique screen names are useful
message| new display text


*service:* rest_command.pixelit_next

switch to next screen. decreases the repeat counter of the actual screen!!

#### Sound

The new [Firmware](https://www.bastelbunker.de/wp-content/uploads/PixelIt.zip) of pixelIt supports mp3 playback via a DFPlayer Module. If you use sound in your templates, the sound is only played once, directly when you add the template to the loop.
Watch Out, the [Awtrix](https://blueforcer.de/awtrix-2-0/) Hardware uses pin D5 for RX to the DFMini, so they are not interchangeable

#### On screen names

You may use screen-names multiple times but if you e.g. delete a screen e.g. "data" all screens of that name will be delete.