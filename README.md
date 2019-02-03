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

### Global regular expressions


#### ss.com property urls
```
Expression type: Result is TRUE, Expression:
mezhciems|purvciems|plyavnieki|teika
```

#### ss.com flats hand over price
```
Expression type: Result is TRUE, Expression:
^(1[0-4][0-9]|[0-9][0-9]|150) .*$

Expression type: Result is FALSE, Expression:
day|week
```

#### ss.com flats hand over sqm
```
Expression type: Result is TRUE, Expression:
^(2[1-9]|3[0-9])$
```

#### ss.com flats hand over type
```
Expression type: Result is TRUE, Expression:
^602
```

#### ss.com flats sell price
```
Expression type: Result is TRUE, Expression:
^(1)?[0-9],
```

#### ss.com flats sell type
```
Expression type: Result is TRUE, Expression:
LT proj|602|M. im.
```

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
#allow ro user ss.com to group zabbix
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
In a template there is defined
```
{#SQM} = ^([0-3][0-9])$
{#URL} = @ss.com property urls
```

## Create host
create a host "ss.com flats hand over" and attach the template "ss.com property"
create a host "ss.com flats sell" and attach the template "ss.com property"

## create a cronjob
```
*/15 * * * * ss.com cd /usr/lib/zabbix/externalscripts && ./ss-com-deliver-json.sh "ss.com flats hand over" https://www.ss.com/en/real-estate/flats/riga/all/hand_over /dev/shm
50 * * * * ss.com cd /usr/lib/zabbix/externalscripts && ./ss-com-deliver-json.sh "ss.com flats sell" https://www.ss.com/en/real-estate/flats/riga/all/sell/ /dev/shm
```

### How it works
All filtering happens through global regular expression section
![Global Regular Expression](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-global-regex.png)

We have a basic filtering in template level:
![Template level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-template.png)

For "hand over" host we got:
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-hand-over.png)

For "sell" host we got:
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-sell.png)

Be carefull when modify template Filters. It will totaly override (delete everything) in host level filtering!


