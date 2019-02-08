# ss.com-property-zabbix

```
{
  "data": [
    {
      "{#URL}": "https://www.ss.com/msg/en/real-estate/flats/riga/ziepniekkalns/dkodg.html",
      "{#PLACE}": "Ziepniekkalns, Kartupelu 16",
      "{#ROOMS}": "2",
      "{#SQM}": "60",
      "{#FLOOR}": "1",
      "{#TYPE}": "Private",
      "{#PRICE}": "330  /mon."
    },
    {
      "{#URL}": "https://www.ss.com/msg/en/real-estate/flats/riga/kengarags/aexgx.html",
      "{#PLACE}": "Kengarags, Maskavas 266",
      "{#ROOMS}": "2",
      "{#SQM}": "47",
      "{#FLOOR}": "3",
      "{#TYPE}": "Chrusch.",
      "{#PRICE}": "420  /mon."
    },
    {
      "{#URL}": "https://www.ss.com/msg/en/real-estate/flats/riga/teika/dplek.html",
      "{#PLACE}": "Teika, Pikola 19",
      "{#ROOMS}": "1",
      "{#SQM}": "35",
      "{#FLOOR}": "2",
      "{#TYPE}": "Perewar",
      "{#PRICE}": "150  /mon."
    }
  ]
}

```

## Dependencies

### Global regular expressions

At first we need to specify the scope of interests


## Regions interested
Set which regions you are interested the most see [flats riga](https://www.ss.com/en/real-estate/flats/riga/)
![Flats Riga](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-riga-regions.png)

In Zabbix this must be configured as:
![com-property-url](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/regex-ss-com-property-url.png)
#### ss.com property urls
```
mezhciems|purvciems|plyavnieki|teika
```

## Price range for renting
![com-property-url](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/regex-ss-com-flats-hand-over-price.png)
What is the price range as 00 - 150. Eliminate paying per "day" or "week"
#### ss.com flats hand over price
```
^(1[0-4][0-9]|[0-9][0-9]|150) .*$
```

Result is FALSE for:
```
day|week
```

## square meters affordable
![com-property-url](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/regex-ss-com-flats-hand-over-sqm.png)
Set the square meters interested. The following regular expression ilustrates range from 21 - 39 square meters.
#### ss.com flats hand over sqm
```
^(2[1-9]|3[0-9])$
```

## Price range for buying
![com-property-url](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/regex-ss-com-flats-sell-price.png)
#### ss.com flats sell price
It must be starting with from 0,000 - 19,000
```
^(1)?[0-9],
```

All together:
![com-property-url](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-global-regex.png)


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

## Fetch and observe "hand over" data 
```
cd /usr/lib/zabbix/externalscripts
./ss-com-property-discover.sh https://www.ss.com/lv/real-estate/flats/riga/all/hand_over > ~/zbx.ss.com.hand.over.json
jq . ~/zbx.ss.com.hand.over.json
```

## Fetch and observe "sell" data
```
cd /usr/lib/zabbix/externalscripts
./ss-com-property-discover.sh https://www.ss.com/lv/real-estate/flats/riga/all/sell > ~/zbx.ss.com.sell.json
jq . ~/zbx.ss.com.sell.json
```

## Install template
Upload [ss.com-property.xml](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-property.xml) to you Zabbix instance

## Create host "ss.com flats hand over"
1) create host with name "ss.com flats hand over"
2) assign template "ss.com property"
3) go to discovery section. Open discovery "Discover all msg items from one section"
4) open filter section, make sure we got
```
{#PRICE} = @ss.com flats hand over price
{#SQM} = @ss.com flats hand over sqm
{#URL} = @ss.com property urls
```
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-hand-over.png)


## Create host "ss.com flats sell"
1) create host with name "ss.com flats sell"
2) assign template "ss.com property"
3) go to discovery section. Open discovery "Discover all msg items from one section"
4) open filter section, make sure we got
```
{#PRICE} = @ss.com flats sell price
{#URL} = @ss.com property urls
```
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-sell.png)


## create global cronjob which will run on behalf user 'ss.com'
```
cat /etc/crontab

# "hand over" changes will be detected every 15 minutes
*/15 * * * * ss.com cd /usr/lib/zabbix/externalscripts && ./ss-com-deliver-json.sh "ss.com flats hand over" https://www.ss.com/en/real-estate/flats/riga/all/hand_over /dev/shm

# Items for selling will be detected every hour
50 * * * * ss.com cd /usr/lib/zabbix/externalscripts && ./ss-com-deliver-json.sh "ss.com flats sell" https://www.ss.com/en/real-estate/flats/riga/all/sell /dev/shm
```



# Summary, how it works
All filtering happens through global regular expression section
![Global Regular Expression](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-global-regex.png)

We have a basic filtering in template level which will take only the regions we are interested:
![Template level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-template.png)

For "hand over" host we got extra definition for square meters and price:
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-hand-over.png)

For "sell" host we got extra definition on price:
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-sell.png)

Be carefull when modify template Filters. It will totaly override (delete everything) in host level filtering!

An event will be generate once the item matched the filer section
![Create dummy item](https://github.com/catonrug/ss.com-property-zabbix/blob/master/ss.com-property-dummy-trapper-item.png?raw=true)

If item is no longer server on ss.com It will get instantly creared from Zabbix
![Get off item from menu](https://github.com/catonrug/ss.com-property-zabbix/blob/master/ss.com-property-item-gets-off-menu.png?raw=true)

## Events
![Events](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-property-in-use.png)

## Action
Condition must match the template name
![Condition match template](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/action-condition-match-template-ss.com-property.png)

Operation
![Operation](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/action-operations.png)


Default subject:
```
Property: {EVENT.NAME}
```
Default message:
```
{TRIGGER.DESCRIPTION}
```

