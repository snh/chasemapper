# Project Horus - Browser-Based HAB Chase Map
Chasemapper is a mapping system designed specifically for chasing high-altitude weather balloons, be it those you might launch yourself, or those launched by your local weather bureau. It is Project Horus's cross-platform replacement for [oziplotter](https://github.com/projecthorus/oziplotter), which was our original offline mapping system, used during many high-altitude balloon flights from 2010 through 2019. 

![ChaseMapper Screenshot](https://github.com/projecthorus/chasemapper/raw/master/doc/chasemapper.jpg)

The primary purpose of chasemapper is to provide an easy-to-use mapping interface to help you as close as possible to the landing location of a high-altitude balloon payload, ideally before the payload gets there so you can watch it land! It does this by providing live predictions of the balloon flight path during the flight, calculated from GFS weather models which are downloaded before you head off on the chase. Maps can also be served up from a local cache, allowing use without internet connectivity (useful here in Australia!). 

Chasemapper is intended to be run on a 'headless' machine like a Raspberry Pi and is accessed from a tablet or laptop computer via a web browser. Multiple clients can connect to the server to see what's going on, which is a nice way of keeping passengers entertained ;-)

It will quite happily run alongside other Project Horus applications such as [radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx/wiki), or [horusdemodlib](https://github.com/projecthorus/horusdemodlib/wiki)

### Contacts
* [Mark Jessop](https://github.com/darksidelemm) - vk5qi@rfhead.net

## Update Notes
* If you have previously had chasemapper or auto_rx installed, you may need to update flask-socketio to the most recent version. You can do this by running `sudo pip3 install -U flask-socketio`
* Chasemapper has now dropped support for the HabHub tracker, which is due to be retired later this year (2022). Chasemapper supports uploading chase-car position information to both [SondeHub](https://tracker.sondehub.org) (for radiosonde chasers), and [SondeHub-Amateur]((https://amateur.sondehub.org)) (for amateur high-altitude balloon launches)

## Docker Install
The fastest (and most recommended) way to get chasemapper up and running is to use the pre-built docker container. Information on using this is available here: https://github.com/projecthorus/chasemapper/wiki/Docker


## 'Local' Install - Dependencies
If you are using Docker, you can skip this section.

**Note: ChaseMapper requires Python 3.6 or newer.**

On a Raspbian/Ubuntu/Debian system, you can get most of the required dependencies using:
```
$ sudo apt-get install git python3-numpy python3-requests python3-serial python3-dateutil python3-flask python3-pip
```
On other OSes the required packages should be named something similar. 

You also need flask-socketio (>=5.0.0) and pytz, which can be installed using pip:
```
$ sudo pip3 install flask-socketio pytz
```

You can then clone this repository with:
```
$ git clone https://github.com/projecthorus/chasemapper.git
```

## Telemetry Sources
To use the map, you need some kind of data to plot on it! The mapping backend accepts telemetry data in a few formats:
* 'Payload Summary', 'Chase Car Position' and 'Bearing' messages, via UDP broadcast in a JSON format [described here](https://github.com/projecthorus/horus_utils/wiki/5.-UDP-Broadcast-Messages#payload-summary-payload_summary). The standard ports used for these are 55672 (for hobbyist HAB payloads) and 55673 (radiosondes). These can be generated by:
  * [radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx/wiki) - See [here](https://github.com/projecthorus/radiosonde_auto_rx/wiki/Configuration-Settings#payload-summary-output) for configuration info.
  * The [Horus-GUI](https://github.com/projecthorus/horus-gui) and [Horus Binary](https://github.com/projecthorus/horusdemodlib/wiki) 4FSK telemetry decoders will emit these messages on port 55672 by default.
  * My [Kraken-SDR fork](https://github.com/darksidelemm/krakensdr_doa) will emit TDOA bearing information in the appropriate UDP message format on port 55672. Note that the bearing functionality is very much experimental at this time.
* 'OziMux' messages, via UDP broadcast in a simple CSV format [described here](https://github.com/projecthorus/oziplotter/wiki/3---Data-Sources#3---oziplotter-data-inputs).
  * [radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx/wiki) - See [here](https://github.com/projecthorus/radiosonde_auto_rx/wiki/Configuration-Settings#sending-payload-data-to-chasemapper--oziplotter--ozimux) for configuration info, though I suggest using the 'Payload Summary' message as described above as it provides callsign information.
  * Pi-in-the-Sky's [lora_gateway](https://github.com/PiInTheSky/lora-gateway) - Using the `OziPort=8942` configuration option.


## Configuration & Startup
Many settings are defined in the [horusmapper.cfg](./horusmapper.cfg.example) configuration file.
Create a copy of the example config file using
```
$ cp horusmapper.cfg.example horusmapper.cfg
```
Edit this file with your preferred text editor. The configuration file is fairly descriptive - you will need to set:
 * At least one telemetry 'profile', which defines where payload and (optionally) car position telemetry data is sourced from.
 * A default latitude and longitude for the map to centre on.

The example configuration file includes profiles suitable for receiving data from radiosonde_auto_rx, and from [Horus-GUI](https://github.com/projecthorus/horus-gui).

Once configured, you can start-up the horusmapper server with:
```
$ python3 horusmapper.py
```

The server can be stopped with CTRL+C. Sometimes the server doesn't stop cleanly and may the process may need to be killed. (Sorry!)

You should then be able to access the webpage by visiting http://your_ip_here:5001/

## Live Predictions
By default, chasemapper will attempt to request flight-path predictions from the SondeHub instance of the [Tawhiri Predictor](https://github.com/projecthorus/tawhiri), which requires an internet connection. If you have a semi-reliable internet connection during the flight, this might be all you need to get chasing!

However, if you think you might be going out of phone coverage range, you may want to set up offline predictions:

### Offline Predictions
To do this you need cusf_predictor_wrapper and it's dependencies installed. Refer to the [documentation on how to install this](https://github.com/darksidelemm/cusf_predictor_wrapper/). If you are using Docker, you can skip this section as it will already be set up.

Once compiled and the python library installed, you will need to: 
 * Copy the 'pred' binary into this directory. If using the Windows build, this will be `pred.exe`; under Linux/OSX, just `pred`.

You will then need to modify the horusmapper.cfg Predictor section setting as necessary to reflect the predictory binary location, the appropriate model_download command.

You can then click 'Download Model' in the web interface's setting tab to trigger a download of the latest GFS model data. Offline predictions will start automatically once a valid model is available. You can tell if you are using Online or Offline predictions by an '(Online)' or '(Offline)' indication next to the 'Current Model' line in the status panel.

## Chase Car Positions
At the moment Chasemapper supports receiving chase-car positions via either GPSD, a Serial-attached GPS, or Horus UDP messages. Refer to the configuration file for setup information for these options.

This application can also plot your position onto the tracker.habhub.org map, so others can see when you're out balloon chasing. You can also fetch positions of nearby chase cars from SondeHub/SondeHub-Amateur, to see if others are out chasing as well :-) These options can be enabled from the control pane on the left of the web interface, and can also be set within the configuration file. 

## Offline Mapping 
Chasemapper can serve up map tiles from a specified directory to the web client. Of course, for this to be useful, we need map tiles to serve! 

Serving of local map tiles can be enabled by setting `[offline_maps] tile_server_enabled = True`, and changing `[offline_maps] tile_server_path` to point to your tile cache directory (i.e. `/home/pi/Maps/`). Chasemapper will assume each subdirectory in this folder is a valid map layer (e.g. `~/Maps/OSM/`, `~/Maps/opencyclemap/`). and will add them to the map layer list at the top-right of the interface.

Note that if you want to use these offline maps within a Docker container, you will need to [modify the tile server path](https://github.com/projecthorus/chasemapper/blob/master/horusmapper.cfg.example#L172) in your configuration file to be /opt/chasemapper/Maps/

### Option 1 - FoxtrotGPS's Tile Cache
Another option to obtain map tiles is [FoxtrotGPS](https://www.foxtrotgps.org/).

To grab map tiles using FoxtrotGPS, we're going to use FoxtrotGPS's [Cached Maps](https://www.foxtrotgps.org/doc/foxtrotgps.html#Cached-Maps) feature. 

 * Install FoxtrotGPS (Linux only unfortunately, works OK on a Pi!) either [from source](https://www.foxtrotgps.org/releases/), or via your system package manager (`sudo apt-get install foxtrotgps`). 
 * Load up FoxtrotGPS, and pan around the area you are intersted in caching. Pick the map layer you want, right-click on the map, and choose 'Map download'. You can then select how many zoom levels you want to cache, and start it downloading (this may take a while!)
 * Once you have a set of folders within your `~/Maps` cache directory, you can startup Chasemapper and start using them! Tiles will be served up as they become available.

### Option 2 - MapTilesDownloader
[MapTilesDownloader](https://github.com/Moll1989/MapTilesDownloader) can be setup on your RPi, allowing access via a web browser to select tile regions. Colin Moll's fork (linked above) includes a systemd service for starting this on boot.

Note that a number of options in this fork are hard-coded, e.g. the [TCP port](https://github.com/Moll1989/MapTilesDownloader/blob/master/src/server.py#L227) it listens on, and the [list of map tiles](https://github.com/Moll1989/MapTilesDownloader/blob/master/src/UI/main.js#L13).

This option still needs some work to be usable, as it won't write directly into the `~/Maps/` directory.

## Running as a Systemd Service
Chasemapper can be operated in a 'continuous' mode, running as a systemd service. I use this in my chase car so that I can power up my car Raspberry Pi, and have services like auto_rx and chasemapper running immediately. 

If you're using docker, this is already sorted out for you, and the docker container will run at startup.

To set this up, the chasemapper.service file  must be edited to include your username, and the path to this directory.

```
$ sudo cp chasemapper.service /etc/systemd/system/
$ sudo nano /etc/systemd/system/chasemapper.service
```

If you are not running chasemapper on a Raspberry Pi as the 'pi' user, you will need to edit the chasemapper.service file and modify
the `ExecStart`, `WorkingDirectory` and `User` fields. Otherwise, leave all settings at their defaults:

```
[Unit]
Description=chasemapper
After=syslog.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/chasemapper/horusmapper.py
Restart=always
RestartSec=3
WorkingDirectory=/home/pi/chasemapper/
User=pi
SyslogIdentifier=chasemapper

[Install]
WantedBy=multi-user.target
```

Once/if edited, install and start the service using:
```
$ sudo systemctl enable chasemapper.service
$ sudo systemctl start chasemapper.service
```

The debug log output can be viewed buy running:
```
$ sudo journalctl -u chasemapper.service -f -n
```

To stop the service, simply run:
```
$ sudo systemctl stop chasemapper.service
```

## Radio Direction Finding Support
As of August 2019, Chasemapper can also plot bearings from radio-direction-finding devices. Bearing information is accepted in the 'horus_udp' format (essentially, JSON over UDP broadcast), and can be provided as either 'relative' (bearing relative to front-of-car, with no source position information), or 'absolute' (bearing relative to true north, with a source lat/lon). Relative bearings will be fused with the instantaneous car heading, which is currently calculated from speed-gated GPS headings.

![Bearings Screenshot](https://github.com/projecthorus/chasemapper/raw/master/doc/bearings.jpg)

The following formats are currently supported:
```
# Absolute bearings - lat/lon and true bearing provided
{'type': 'BEARING', 'bearing_type': 'absolute', 'latitude': latitude, 'longitude': longitude, 'bearing': bearing}

# Relative bearings - only relative bearing is provided.
{'type': 'BEARING', 'bearing_type': 'relative', 'bearing': bearing}

The following optional fields can be provided:
    'source': An identifier for the source of the bearings, i.e. 'kerberos-sdr', 'yagi-1'
    'timestamp': A timestamp of the bearing provided by the source.
    'confidence': A confidence value for the bearing, from 0 to [MAX VALUE ??]
    'power': A reading of signal power
    'raw_bearing_angles': A list of angles, associated with...
    'raw_doa': A list of TDOA result values, for each of the provided angles.
```

The above formats are accepted via a horus_udp listener, and so you must have a [profile](https://github.com/projecthorus/chasemapper/blob/master/horusmapper.cfg.example#L18) set up with a `telemetry_source_type` of `horus_udp`. 

Bearings are plotted on the map as thin lines, which slowly become transparent as they get older, and then disappear. The style of the line and the maximum age bearings shown can be configured in the new bearing settings tab on the left of the screen (click the compass icon). You can also filter bearings by the optionally supplied confidence level ('Confidence Threshold'). Bearings provided while the chase-car is stationary (i.e. when the heading is essentially unknown) are filtered out of the display by default, but can be enabled if desired ('Show stationary bearings'). Most of the filter settings will only take effect by clicking the 'Redraw Bearings' button.

My [Kraken-SDR fork](https://github.com/darksidelemm/krakensdr_doa) will emit relative bearings in the above format on UDP port 55672, including the raw TDOA data, which is plotted on a polar plot on the bottom-right of the display. Bearing data will be emitted as soon as TDOA processing is started. Note that I have only tested with data from a Uniform Circular Array and do not currently handle forward/reverse ambiguities from a linear array configuration. I would *not* suggest running Chasemapper on the same device as the Kerberos-SDR software, due to the high processor load of the Kerberos algorithms.

Note that the bearing display (in particular the TDOA data polar plot) does put a fairly big strain on some slower devices. Currently the polar plot is generated in a fairly naive way, and definitely has room for improvement. 

I make no promises as to the usefulness and/or performance of this feature in chasemapper - it's essentially a re-implementation of a radio-direction finding mapping system developed by fellow Amateur Radio Experimenters Group members a very long time ago. I've used it in a few local amateur radio direction finding competitions have found it to be useful. It's also important to note that attempting to direction-find radiosonde/high-altitude balloon payloads which are located at high relative elevations (>40 degrees or so) is likely to lead to very inaccurate results due to coning angle limitations (where a bearing cannot be resolved due to insufficient phase-delta between receive antennae). 