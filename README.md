ss.com-property-zabbix
======================

{
  "data": [
    {
      "{#URL}": "https://www.ss.com/msg/lv/real-estate/flats/riga/kengarags/gdkej.html",
      "{#PLACE}": "engarags, Salaspils 7",
      "{#ROOMS}": "2",
      "{#SQM}": "34",
      "{#FLOOR}": "5",
      "{#TYPE}": "ehu pr.",
      "{#PRICE}": "240  /mn."
    },
    {
      "{#URL}": "https://www.ss.com/msg/lv/real-estate/flats/riga/mangalsala/addpd.html",
      "{#PLACE}": "Mangasala, Albatrosu 24",
      "{#ROOMS}": "2",
      "{#SQM}": "52",
      "{#FLOOR}": "5",
      "{#TYPE}": "Specpr.",
      "{#PRICE}": "390  /mn."
    },
    {
      "{#URL}": "https://www.ss.com/msg/lv/real-estate/flats/riga/katlakalns/gjgex.html",
      "{#PLACE}": "Katlakalns, Bauskas 1",
      "{#ROOMS}": "2",
      "{#SQM}": "68",
      "{#FLOOR}": "1",
      "{#TYPE}": "Hru.",
      "{#PRICE}": "300  /mn."
    }
  ]
}

Dependencies
------------

* Install external script
cd /usr/lib/zabbix/externalscripts
curl https://raw.githubusercontent.com/catonrug/externalscripts/master/ss-com-property-discover.sh > ss-com-property-discover.sh
chmod 770 ss-com-property-discover.sh
chown zabbix. ss-com-property-discover.sh

* Test script
./ss-com-property-discover.sh https://www.ss.com/lv/real-estate/flats/riga/all/hand_over > ~/hand.over.json

* Install jq utility to beautifull the output
apt install jq # on ubuntu, debian
curl -sL https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64 -o /usr/bin/jq && chmod +x /usr/bin/jq # on centos

* test recieved content
jq . ~/hand.over.json

