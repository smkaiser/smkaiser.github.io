# PiAware to Azure

## Notes

PiAware basic build: https://flightaware.com/adsb/piaware/build

Stream location info to file: 

`wget -O - -q http://localhost:30003  log.txt`

`tail -f log.txt | grep "MSG,[134]"`

Output format looks like:

```
MSG,3,1,1,A58C2E,1,2017/02/18,10:33:40.304,2017/02/18,10:33:40.339,,4975,,,47.61324,-122.42848,,,,,,0
MSG,7,1,1,A8103E,1,2017/02/18,10:33:40.374,2017/02/18,10:33:40.396,,7300,,,,,,,,,,
MSG,7,1,1,A8103E,1,2017/02/18,10:33:40.518,2017/02/18,10:33:40.557,,7300,,,,,,,,,,
MSG,7,1,1,A58C2E,1,2017/02/18,10:33:40.596,2017/02/18,10:33:40.615,,4950,,,,,,,,,,
MSG,7,1,1,A8103E,1,2017/02/18,10:33:40.603,2017/02/18,10:33:40.615,,7275,,,,,,,,,,
MSG,8,1,1,A58C2E,1,2017/02/18,10:33:40.604,2017/02/18,10:33:40.616,,,,,,,,,,,,0
MSG,3,1,1,C036B1,1,2017/02/18,10:33:40.751,2017/02/18,10:33:40.777,,31350,,,47.97528,-122.64436,,,,,,0
```

Columns in stream from port 30003:

Column | Description
--- | ---
1 | `MSG`
2 | Message Type. Only care about rows where this column = 1, 3, or 4. 
3 | `1`
4 | `1`
5 | Aircraft ICAO code
6 | `1`
7-8 | Date, Time
9-10 | Another Date, Time
11 | Flight or tail number (when type=1)
12 | Altitude (when type=3)
13 | Speed in knots (when type=4)
14 | Heading (when type=4)
15 | Latitude (when type=3)
16 | Longitude (when type=3)
17-22 | Disregard


Script to stream from port 30003, find message types 1/3/4, and post each line to node-red:
```
nc localhost 30003 | grep --line-buffered "MSG,[134]" | while IFS= read -r line; do
        curl -d "body=$line" http://10.0.1.103:1880/adsb
done
```
## Node-red Notes

CSV node columns:
`msg,type,col3,col4,icao,col6,date1,time1,date2,time2,callsign,altitude,speed,heading,latitude,longitude,col17,col18,col19,col20,col21,col22`


