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


#### ss.com property urls
set which regions you are intereseted the most
```
Expression type: Result is TRUE, Expression:
mezhciems|purvciems|plyavnieki|teika
```

#### ss.com flats hand over price
What is the price range, eliminate paying per "day" or "week"
```
Expression type: Result is TRUE, Expression:
^(1[0-4][0-9]|[0-9][0-9]|150) .*$

Expression type: Result is FALSE, Expression:
day|week
```

#### ss.com flats hand over sqm
Set the square metters intereseted
```
Expression type: Result is TRUE, Expression:
^(2[1-9]|3[0-9])$
```

#### ss.com flats sell price
Selling price
```
Expression type: Result is TRUE, Expression:
^(1)?[0-9],
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
./ss-com-property-discover.sh https://www.ss.com/lv/real-estate/flats/riga/all/sell > ~/zbx.ss.com.sell.json
```

## Output on screen
```
jq . ~/zbx.ss.com.hand.over.json
jq . ~/zbx.ss.com.sell.json
```

## Install [ss.com-property.xml](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-property.xml)
In a template there is defined
```
{#URL} = @ss.com property urls
```

## Create host "ss.com flats hand over"
1) "ss.com flats hand over"
2) assign template "ss.com property"
3) go to discovery section. Open discovery "Discover all msg items from one section"
4) open filter section, add fiters
```
{#PRICE} = @ss.com flats hand over price
{#SQM} = @ss.com flats hand over sqm
{#URL} = @ss.com property urls
```
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-hand-over.png)


## Create host "ss.com flats sell"
1) "ss.com flats sell"
2) assign template "ss.com property"
3) go to discovery section. Open discovery "Discover all msg items from one section"
4) open filter section, add fiters
```
{#PRICE} = @ss.com flats sell price
{#URL} = @ss.com property urls
```
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-sell.png)


## create global cronjob which will run on behalf user 'ss.com'
```
cat /etc/crontab

# hand out changes will be detected every 15 minutes
*/15 * * * * ss.com cd /usr/lib/zabbix/externalscripts && ./ss-com-deliver-json.sh "ss.com flats hand over" https://www.ss.com/en/real-estate/flats/riga/all/hand_over /dev/shm

# properties for selling will be detected every hour
50 * * * * ss.com cd /usr/lib/zabbix/externalscripts && ./ss-com-deliver-json.sh "ss.com flats sell" https://www.ss.com/en/real-estate/flats/riga/all/sell /dev/shm
```

# Summary, how it works
All filtering happens through global regular expression section
![Global Regular Expression](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/ss.com-global-regex.png)

We have a basic filtering in template level:
![Template level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-template.png)

For "hand over" host we got:
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-hand-over.png)

For "sell" host we got:
![Host level filtering](https://raw.githubusercontent.com/catonrug/ss.com-property-zabbix/master/filters-sell.png)

Be carefull when modify template Filters. It will totaly override (delete everything) in host level filtering!


