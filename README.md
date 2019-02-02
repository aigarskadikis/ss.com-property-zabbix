# ss.com-property-zabbix

```
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
```

## Dependencies

### Install external script
```
cd /usr/lib/zabbix/externalscripts
curl https://raw.githubusercontent.com/catonrug/externalscripts/master/ss-com-property-discover.sh > ss-com-property-discover.sh
curl https://raw.githubusercontent.com/catonrug/externalscripts/master/ss-com-deliver-json.sh > ss-com-deliver-json.sh
chmod 770 ss-com-property-discover.sh
chmod 770 ss-com-deliver-json.sh
chown zabbix. ss-com-property-discover.sh
chown zabbix. ss-com-deliver-json.sh
```

### Install jq utility on ubuntu, debian, raspiban
```
apt install jq
```
### Install jq utility on CentOS, RHEL
```
curl -sL https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64 -o /usr/bin/jq && chmod +x /usr/bin/jq
```

### Install other linux user "ss.com" which will be responsible about task
```
groupadd ss.com
useradd -s /sbin/nologin -g ss.com ss.com
usermod -a -G zabbix ss.com
grep ss.com /etc/passwd
id ss.com
chmod -R 770 /usr/lib/zabbix/externalscripts/*
```

## Fetch the "hand over" data 
```
cd /usr/lib/zabbix/externalscripts
./ss-com-property-discover.sh https://www.ss.com/lv/real-estate/flats/riga/all/hand_over > ~/zbx.ss.com.hand.over.json
```

## Fetch the "sell" data
```
cd /usr/lib/zabbix/externalscripts
./ss-com-property-discover.sh https://www.ss.com/lv/real-estate/flats/riga/all/sell/ > ~/zbx.ss.com.sell.json
```

## Output on screen
```
jq . ~/zbx.ss.com.hand.over.json
jq . ~/zbx.ss.com.sell.json
```

## Install template
https://github.com/catonrug/ss.com-property-zabbix/blob/master/ss.com-property.xml

## Create host
create a host "ss.com flats hand over" and attach the template "ss.com property"
create a host "ss.com flats sell" and attach the template "ss.com property"



