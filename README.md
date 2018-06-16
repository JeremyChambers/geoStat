# geoStat
**Geo/Meteorological Analytics with Linked Data**

**Goal:** Return a sorted list of the "Wettest" MSAs (metropolitan statistical areas) in May, 2015.  

* The **population wetness** of an MSA is calculated as the **MSA population** times the **amount of rain** received.  
* Assume that all people remain inside between the hours of 12 AM and 7 AM local time and so rainfall during these hours does not count. 

**Ensure:**
- [x] capture project to gitHub :octocat:
- [ ] program correctness
- [ ] code style 
- [ ] documentation quality
- [ ] packaging quality
- [ ] performance / efficiency
- [ ] portability (Docker, Vagrant, etc.)
- [ ] security
- [ ] bonus points will be given for visualizations of the data (charts, graphs, etc.)

## Resources
http://www.ncdc.noaa.gov/data-access/quick-links#loc-clim
http://www.ncdc.noaa.gov/orders/qclcd/QCLCD201505.zip
http://www.ncdc.noaa.gov/homr/file/wbanmasterlist.psv.zip;jsessionid=A76E5C1AC28E5985D6EFC66A156DF6BE
https://www.census.gov/data/tables/2016/demo/popest/total-metro-and-micro-statistical-areas.html

## Example Metropolitan Statistical Area plot (Feb 2013)

![MSA Example Plot from 2013](https://www2.census.gov/geo/maps/metroarea/us_wall/Feb2013/cbsa_us_0213_large.gif)

### Definition of MSA
The Federal Office of Management and Budget (OMB) designates and defines.  

[Census.gov : See Chapt 13, Metropolitan Areas: MA, MSA, PMSA, CMSA, NECMA](https://www2.census.gov/geo/pdfs/reference/GARM/Ch13GARM.pdf)

* [Feb 2013 OMB Bulletin](https://www.whitehouse.gov/sites/whitehouse.gov/files/omb/bulletins/2013/b-13-01.pdf) MSA delineation update (*381 MSA*s for USA)
  * this MSA delineation _should be_ the one in effect for our historical frame of reference (May 2015).  Not sure if currently available historical data from census.gov reflects the temporally evolving MSA delineations or not.
  * Delineation _files_: https://www.census.gov/geographies/reference-files/time-series/demo/metro-micro/delineation-files.html
    * Feb 2013 : Core based statistical areas (CBSAs), metropolitan divisions, and combined statistical areas (CSAs).  Includes FIPS codes for State and County.  County is the linkable element to the NOAA stations with curated rain data
    * 2015 : KML file (i.e., for Google Earth viz of MSA's) https://www.census.gov/geo/maps-data/data/kml/kml_msa.html
* [Jul 2015 OMB Bulletin](https://www.whitehouse.gov/sites/whitehouse.gov/files/omb/bulletins/2015/b-15-01.pdf) MSA delineation update (*382 MSA*s for USA)
* [Aug 2017 OMB bulletin](https://www.whitehouse.gov/sites/whitehouse.gov/files/omb/bulletins/2017/b-17-01.pdf) MSA delineation update (*383 MSA*s for USA)

#### Definition of FIPS
FIPS are unique IDs for _counties_ and county equivalents in the United States, certain U.S. possessions, and certain freely associated states.
U.S. Census Bureau, while keeping the name (FIPS), replaced the FIPS 6-4 codes with [INCITS 31](https://standards.incits.org/apps/group_public/project/details.php?project_id=204) codes after the 2010 Census, with the Census bureau assigning new codes as needed for their internal use during the transition. -- https://en.wikipedia.org/wiki/FIPS_county_code

* [May, 2015 County Changes](https://www.census.gov/geo/reference/county-changes.html)

## Approach

* Research concepts such as MSA and how to link it with rainfall data from potentially multiple NOAA sources within each MSA
* Prepare population data for each MSA on May 2015
* For each MSA, determine the _contained_ set of WBAN reporter stations
  * Currently searching for this linked set of data.  Worse case, have to do a point-in-polygon test using longitude/latitude coords versus MSA polygon whose vertexes are longitude/latitude projections.
  * [Cran R Project that already did this work](https://cran.r-project.org/web/packages/countyweather/vignettes/countyweather.html)
* Prepare _timezone aware_ rainfall data for each MSA on May 2015, less the _localized_ exclusion window (12 AM - 7 AM)
  * If this data doesn't already exist, have to do a weighted average of WBAN station recordings (and interpolate missing values)

## Pseudo code

```java
    
    // pseudo schema / code (hack)
    
    class MSA {
       string name;
       int    populationSize = 0; // on May 2015 
       float  populationWetness = 0.0;
       
       // TODO: might need this data in lieu of existing MSA:WBAN membership info
       Polygon< Pair<float latitude, float longitude> > perimeter;
       
       // int timeZoneOffset; ... Even if a MSA spans multiple timezones, timeZoneOffset not relevant 
       //                         because we record data indexed to local timezone [0..23]
       
       class WBAN {
          string name;
          string parent = this.name;
          float  latitude;
          float  longitude;
          
          // May 2015 rain observations (local timezone)
          // Note: -1 == missing observation
          float hourlyRainObservation[24]; // 0 == 12 to 12:59 AM, 1 == 1 AM to 1:59 AM, ... 23 == 23 to 23:59 AM
          
          return float observedRainfall( startHour, endHour ) {
             assert startHour >= 0;
             assert endHour < 24;
             assert startHour <= endHour;
             
             hourlyRainObservation = map( dataRepairFunctor, hourlyRainObservation, startHour, endHour);
             
             return map( sumFunctor, hourlyRainObservation, startHour, endHour );
          }
       } // class WBAN
       
       // List of WBANs inside this MSA
       List<WBANinfo> wban;
       
    } // class MSA
```

``` Java

    // load all US MSA info and their contained WBAN observational data for May 2015
    msaList = loadMSAs();

    // update population wetness scores for each MSA
    //
    const int startHour = 7;
    const int endHour = 23;
    for each msa in msaList {
        msaSumRainfall = 0.0;
        for each wban in msa.WBAN {
           msaSumRainfall += wban.observedRainfall(startHour, endHour);
        }
        
        // initially just average the WBANs observations inside the MSA
        float observedRainfallAvg = msaSumRainFall / msa.WBAN.size();
        
        msa.populationWetness = observedRainfallAvg * msa.populationSize;
     } 
     
     // Sort : could be done with iterable over a sortable derived class
     //
     boolean ascending = false;     
     sortMSAbyWettestPopulation( msaList, ascending ); 
     
     // then dump list
     
``` 

**Data Sources**

* MSA : 

* QCLCD201505/201505station.txt ($ head -n 10, tail -n 10)

WBAN|WMO|CallSign|ClimateDivisionCode|ClimateDivisionStateCode|ClimateDivisionStationCode|Name|State|Location|Latitude|Longitude|GroundHeight|StationHeight|Barometer|TimeZone
--|--|--|--|--|--|--|--|--|--|--|--|--|--|--
00100||M89||03||ARKADELPHIA|AR|DEXTER B FLORENCE MEM FLD AP|34.09972|-93.06583|182|||-6
00101||KQHT||||BISHKEK||MANAS INTERNATIONAL AIRPORT|43.067|74.483|2090|||6
00102||IAN||50||KIANA|AK|BOB BARKER MEMORIAL AIRPORT|66.983|-160.433|168|||-9
00103||IWK||50||WALES|AK|WALES AIRPORT|65.617|-168.1|23|||-9
00104||FSP||50||NIKOLAI|AK|NIKOLAI AIRPORT|63.017|-154.367|414|||-9
00105||MDM||50||MARSHALL|AK|MARSHALL DON HUNTER SR AIRPORT|61.867|-162.033|102|||-9
00106||IIK||50||AKIAK|AK|KIPNUK AIRPORT|60.9|-161.233|30|||-9
00107||KNW||50||NEW STUYAHOK|AK|NEW STUYAHOK AIRPORT|59.45|-157.333|302|||-9
00108||SCM||50||SCAMMON BAY|AK|SCAMMON BAY AIRPORT|61.85|-165.567|14|||-9
96401||IEC||50||CHULITNA|AK|CHULITNA|62.82639|-149.90667|1400|1350|0|-9
96402||JVM|05|50|8915|SUTTON|AK|JONESVILLE MINE AIRPORT|61.7138|-148.9088|550|870|0|-9
96404|70292|63D0|08|50||TOK|AK|TOK 70 SE|62.737|-141.2083|2000|||-9
-|91355||04|91|||FM|KOSRAE|5.35|162.95|7|||12
-|91425||04|91|4590||FM|NUKUORO|3.85|155.01667|8|||11
-|91369||04|91|4304||MH|JALUIT|5.91667|169.65|6|||12
-|91339||04|91|4419||FM|LUKUNOCH|5.51667|153.81667|5|||10
-|91371||04|91|4903||MH|WOTJE|9.46667|170.25|6|||12
-|91258||04|91|4895||MH|UTIRIK|11.23333|169.85|6|||12
-|91367||04|91|3915||MH|AILINGLAPALAP|7.26667|168.83333|6|||12

* observations:
  * GOOD data: links WBAN to state, timezone, clean lat/longitude 

* QCLCD201505/201505Hourly.txt : ($ cut -d',' -f1-5,9,23,39,40,41,42 --output-delimiter='|' QCLCD201505/201505hourly.txt | head -n 10)

WBAN|Date|Time|StationType|SkyCondition|WeatherType|RelativeHumidity|RecordType|RecordTypeFlag|HourlyPrecip|HourlyPrecipFlag
--|--|--|--|--|--|--|--|--|--|--
00102|20150501|0001|0|CLR| | 88|AA| | |
00102|20150501|0101|0|CLR| | 88|AA| | |
00102|20150501|0201|0|CLR| | 92|AA| | |
00102|20150501|0301|0|CLR| | 92|AA| | |
00102|20150501|0401|0|CLR| | 96|AA| | |
00102|20150501|0501|0|CLR| | 96|AA| | |
00102|20150501|0601|0|FEW060| | 92|AA| | |
00102|20150501|0701|0|CLR| | 92|AA| | |
00102|20150501|0801|0|CLR| | 78|AA| | |

* observations:
  * no precip data, no timezone, only "SkyCondition" seems useful


* wbanmasterlist.psv :

"REGION"|"WBAN_ID"|"STATION_NAME"|"STATE_PROVINCE"|"COUNTY"|"COUNTRY"|"EXTENDED_NAME"|"CALL_SIGN"|"STATION_TYPE"|"DATE_ASSIGNED"|"BEGIN_DATE"|"COMMENTS"|"LOCATION"|"ELEV_OTHER"|"ELEV_GROUND"|"ELEV_RUNWAY"|"ELEV_BAROMETRIC"|"ELEV_STATION"|"ELEV_UPPER_AIR"
---------|-------|------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----
"240/940"|"00000"|"WOLF POINT"|"MT"||"US"||"OLF"|"FAA EXP"|"06/15/2010"||"THIS WBAN WAS ASSIGNED 00000 WAS CORRECTED TO 94017"|"48*05'40""N 105*35'28""W"||||||
"000"|"00100"|"DEXTER B FLORENCE MEMORIAL FIELD AP"|"AR"||"US"||"M89"|"METAR"|"09/21/2012"|"8/1/1960"|"ASSIGNED FOR METAR LOAD FOR QCLCD/ISD"|"34 05 59 N/ -93 03 57 W"||"182 FT"||||
"001"|"00101"|"MANAS INTERNATIONAL AIRPORT"|||"KG"|"MANAS INTERNATIONAL AIRPORT"||"AWOS"|"November 2013"|"01-JAN-01"|"ADDED TO REPOSITORY USING ISD, AIRNAV.COM, FAA PUBLICATIONS AS SOURCE FOR AWOS/AWSS/METAR LOAD PROJECT.  NCDC RECEIVING DATA ON GTS FOR THESE STATIONS. WBANS WERE AUTO-GENERATED STARTING AT 00100."|"43.067, 74.483"||"2090 FEET"||||
"001"|"00102"|"BOB BARKER MEMORIAL AIRPORT"|"AK"|"NORTHWEST ARCTIC BOROUGH"|"US"|"BOB BARKER MEMORIAL AIRPORT"|"IAN"|"AWOS"|"November 2013"|"01-JUL-58"|"ADDED TO REPOSITORY USING ISD, AIRNAV.COM, FAA PUBLICATIONS AS SOURCE FOR AWOS/AWSS/METAR LOAD PROJECT.  NCDC RECEIVING DATA ON GTS FOR THESE STATIONS. WBANS WERE AUTO-GENERATED STARTING AT 00100."|"66.983, -160.433"||"168 FEET"||||

* observations:
  * don't extract lat / longitude data from this .. non-uniform format

