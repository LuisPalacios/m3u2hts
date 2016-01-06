m3u2hts
=======

Genera MUXES y CANALES en Tvheadend 3.9 partiendo de un fichero playlist .M3U. Este fork parte del trabajo Gregor Rudolf (Fuente en [GitHub grudolf/m3u2hts](https://github.com/grudolf/m3u2hts/blob/master/m3u2hts.py)) y está especialmente probado para funcionar con la lista de programas de Movistar TV

Uso
-----

Usar ``m3u2hts.py [opciones] inputfile`` or ``m3u2hts.py -h`` for help with parameters.
If you want to import the configuration directly, you should stop the TVHeadend service and delete current config first. Ubuntu example::

    sudo service tvheadend stop
    sudo su hts
    cd ~/.hts/tvheadend/
    rm iptvservices/* channels/* channeltags/* epggrab/xmltv/channels/*
    m3u2hts.py inputfile.m3u
    exit
    sudo service tvheadend start

Alternatively, run the script somewhere else and transfer the files to TVHeadend config dir when service isn't running.


Input
-----

Channel and optional tag definitions are read from a M3U playlist file::

    #EXTINF:duration,[channel number - ]channel name
    #EXTTV:tag[,tag,tag...];language;XMLTV id[;icon URL]
    udp://@ip:port

The #EXTTV line and its contents are optional.

sample.m3u::

    #EXTINF:0,1 - SLO 1
    #EXTTV:nacionalni;slovenski;SLO1
    udp://@239.1.1.115:5000

    #EXTINF:0,SLO 1 HD
    #EXTTV:nacionalni,hd;slovenski;SLO1;http://cdn1.siol.tv/logo/93x78/slo2.png
    udp://@239.10.2.56:5000


Output
------

The script creates iptvservices, channels, channeltags and epggrab/xmltv/channels directories into which the
following files are written::

    #one file per channel:
    iptvservices/iptv_X
    {
        "pmt": 0,
        "channelname": "SLO 1",
        "port": "5000",
        "interface": "eth1",
        "group": "239.1.1.115",
        "mapped": 1,
        "pcr": 0,
        "disabled": 0
    }

    #one file per channel:
    channels/X
    {
        "name": "SLO 1",
        "xmltv-channel": "SLO1",
        "tags": [
                 1,2
        ],
        "dvr_extra_time_pre": 0,
        "dvr_extra_time_post": 0,
        "channel_number": 1,
        "icon": ""
    }

    #one file per tag:
    channeltags/X
    {
        "enabled": 1,
        "internal": 0,
        "titledIcon": 0,
        "name": "nacionalni",
        "comment": "",
        "icon": "",
        "id": 1
    }
    
    #one file per channel with XMLTV id
    epggrab/xmltv/channels/X
    {
        "channels": [
            1
        ], 
        "name": "SLO1"
    }

Output (TVHeadend 3.9+)
-----------------------

Experimental, use ``--newformat`` switch to create TVHeadend 3.9 compatible configuration.
The file structure is:

    input/iptv/config
    input/iptv/networks/UUID/config                      - one network
    input/iptv/networks/UUID/muxes/UUID/config           - one mux per channel
    input/iptv/networks/UUID/muxes/UUID/services/UUID    - one fake service per channel, -o service option
    channel/tag/UUID                                     - channel tags
    channel/config/UUID                                  - channel (linked to services, tags), -o channel option
    epggrab/xmltv/channels/UUID                          - EPG info (linked to channel), -o channel option

Unfortunately, M3U file is missing a few pieces of information Tvheadend now requires in order to form the mux -> service -> channel chain, most importantly the service id.
The service that m3u2hts creates when you use ``-o service`` option sadly won't work in most cases and you'll have to scan muxes with tvheadend - open network properties, enable "Idle scan muxes" and limit "Max input streams" to something your network can handle.
When the scanning is finished you'll have to map services to channels in either services or channels screen.

If you're lucky and your IPTV provider uses one service per mux and identical names for muxes, services, channels and EPG info, everything will go smoothly.
OTOH, if your provider has 50+ services on a mux for a single working channel and wildly mismatched names, you're better off if you let m3u2hts create channels and optional EPG info with ``-o channel`` option and manually link them to scanned services.


Licencia
--------
Este código se licencia bajo MIT: http://www.opensource.org/licenses/mit-license.php
