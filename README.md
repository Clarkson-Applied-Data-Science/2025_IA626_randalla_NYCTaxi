
## Date Range
The date range was determined using the trip pickup times.  Each row was converted to a DT object and compared against previous entries to determine the most recent and most historic values.  Default values in the past/future were used to initialize the values.  The dates could also have been evaluated based on the drop-off time to get some more recent items, though this effort was focused on the pickup times.

    The file has 13,971,118 lines.
    The latest date is : 2013-12-31 23:59:57
    The earliest date is : 2013-12-01 00:00:00

## Field Names
The data file has an included header line where the data field names were retrieved.  They are shown below with an example value and a brief description.
    'medallion': 'C13BD886F362AC4D71FEB912091AB76A'         These are permits for taxi companies to operate in New York City.  Each one is a unique number and is transferable to others for a fee.
    'hack_license': '3FC36031EAC645D019BAC9C263F21CF7'      This is the license for the particular driver to operate in New York City.
    'vendor_id': 'VTS'                                      This indicates which provider provided the record of the trip.
    'rate_code': '1'                                        The code for the charging amount.
    'store_and_fwd_flag': ''                                This code is used if the taxi had to store the trip data before sending due to poor connectivity.
    'pickup_datetime': '2013-12-30 14:14:00'                The date-time of the pickup.
    'dropoff_datetime': '2013-12-30 14:37:00'               The date-time of the dropoff.
    'passenger_count': '2'                                  How many passengers were in the taxi for this trip.
    'trip_time_in_secs': '1380'                             How long the trip took in seconds.
    'trip_distance': '7.08'                                 The trip distance in miles.
    'pickup_longitude': '-74.001427'                        The longitude coordinate at the pickup location.
    'pickup_latitude': '40.746643'                          The latitude coordinate at the pickup location.
    'dropoff_longitude': '-73.952324'                       The longitude coordinate at the dropoff location.
    'dropoff_latitude': '40.826633'                         The latitude coordinate at the dropoff location.

## Filtering outliers and invalid data 
<a name ="filtering"></a>
Several iterations of filtering were attempted and an effort was made to generalize the solution.  Initially, a fixed bounding of lat/lon coordinates was used to force all trips to occur within a given geographic area.  This is somewhat difficult to apply more generally, since new unique coordinates would be required each time.

Another angle was to use the trip distance and compare it to the lat/lon havershine distances.  This removes items where the lat/lon values and the trip_distance values do not align.  Some of the trips included in this data set appear to occur outside of the geographic region or have invalid lat/lon values and caused outliers to persist through the filtering.  Some trips occured only within Tennessee and some others occured via boats in the middle of the Atlantic Ocean apparently.  Since this effort was focused around NYC Taxi trips, these can be ignored.

The end solution was to use a fixed geographic point and use a radius around it to filter trips that did not start within a reasonable distance.  In this case a 100km radius was used around central NYC coordinates gathered from wikipedia.  This solution allows more options in the future to apply a similar code structure to other geographic areas without requiring major code revisions.

![another one](/NYC_Radius.png)

## Geographic Area
The geographic area plot is provided below.  A more indepth description of the filtering method is provided in [here](#filtering).   The area generally aligns with the tri-state area that would be typical of a NYC taxi trip.

The highest lat/long  :  41.545189 | -72.347534
The lowest lat/long   :  39.751335 | -75.280258

![A plot of the coordinates for the large dataset](/GeographicArea.png)


## Trip Distances
Trip distances were binned to a variety of distances and plotted to show the number of occurences.  The bins were generated iterating through the list in a loop until the value was no longer greater than the list value selected the appropriate bin and updated a dictionary with the counts.  A bins variable was used in case the values for the bins needs to be adjusted.  Results were then displayed using a barplot.  Most of the trips are focused on the lower end of the distance scale.
Example code is shown below.
```
tripDistanceBins = [0.5,.75,1,1.5,2,3,4,5,10,25]
for i in tripDistanceBins:
    if float(line[9]) <i:
        distBin[i]+=1
        break
    if i == len(tripDistanceBins):
        distBin[i] +=1
```
![Trip Distancec Histogram](/TripDistanceHistogram.png)


## Min and Max Values
Multiple approaches to caching the min and max values were used.  An improvement on these would be to generate methods that can be used in many spots without duplicating so much code. 

One method used 2 variables to track the item.  This was initially used for debugging and troubleshooting, but ended up being effective enough to use for the data gathering.

```
if currentDate>cacheDateH:
    cacheDateH=currentDate
if currentDate<cacheDateL:
    cacheDateL=currentDate
```

The next method used a list to compact this somewhat.  Many lists were still generated to support this.  The overall implementation was somewhat cleaner and easier to keep track of.  Ultimately some type of dictionary of lists would have been more preferred.
```
if float(line[9]) > tripDistMinMax[1]: tripDistMinMax[1]=float(line[9])
if float(line[9]) < tripDistMinMax[0] and float(line[9]) > 0: tripDistMinMax[0]=float(line[9])
```
The min and max values are shown below.
    Distance min: 0.01 -- max: 100.0
    Time min: 1.0 -- max: 10800.0
    Passengers min: 1.0 -- max: 9.0

## Unique Values
The categorical items all have varying amounts of unique values.  The medallion and license columns contain a very large number of unique values and are not included in this summary.  A dictionary is provided in the code for review if required.

To genrate the dictionaries a small definition was used to consolidate some of the lines of code.  This is used to check if a key exists and to generate a new key if it isn't found.  The numbers of occurences of these keys is also stored for future review.

```
def checkDict(key,dict):
    if key not in dict:
        dict[key]=1
    else: dict[key]+=1
```

These items were smaller in size and the unique values are included.  There are 2 vendors that provide this service.  Rates are unique to different areas and relate back to a lookup table that NYC provides.  This dataset includes 14 different rate codes.

Vendors:
    {'VTS': 7057292, 'CMT': 6913826}
Rates:
    {'1': 13638315, '2': 257599, '3': 24971, '5': 41889, '4': 4602, '0': 3537, '8': 5, '6': 180, '13': 2, '210': 8, '7': 6, '9': 2, '65': 1, '15': 1}

## Average Travel Distance

Average travel distance was calculated using the provided Haversine distance calculator.  A running tally of trips and total distance was generated and the average caclculated at the end.  Trips of 0 distance reported were excluded from this summation.

The data showed a total distance of 28,752,009.93686609 miles acrossed 13,432,221 trips for an average distance per trip of 2.140525378257705 miles

## MySQL Data Types
For the various fields, options are given for MySQL data types.  Most of self evident for which data type would be appropriate.  Vendor ID included all use 3 characters, though an additional character would give some assurance that future items wouldn't cause an overflow.  The store_and_fwd_flag reports Y/N and could be bool or varchar(1) depending on the required implementation.

    medallian - varchar(32)
    hack_license - varchar(32)
    vendor ID - varchar(4)
    rate code - int
    store_and_fwd_flag - bool or varchar(1)
    pickup_datetime - datetime
    dropoff_datetime - datetime
    passenger_count - int
    trip_time_in_secs - int
    trip_distance - decimal(10,2)
    lat/long - decimal(8,6)

## Sampled Data
To generate a sampled dataset, an if statement arrangement was used.  The Modulous operator provides a convenient way to check things in an orderly way.  For the purposes of this exercise a hard coded value of 1000 was used, though a variable could also be utilized.  Every 1000 rows, a new line is written to a blank CSV and saved to disk for review later.

```
if counter % 1000 == 0: #write a subset for every 1000 rows.
    writer.writerow(line)
```


## Passengers per hour
The average passengers per hour was computed for the entire data set.  This means that all hours across the entire range were summarized together and averaged after full review.  This can be interpretted many ways, for the purposes of this review, the hours were analyzed together for all days.

A code snippet shows the implementation.  A blank dictionary with each hour of the day included is generated.  Using the datetime object, the hour was used to update the appropriate dictionary item when a new trip was found.  The number of trips and the number of passengers was saved and used for the final computation.

```
dateDict = {k:[0,0] for k in range(0,24)}
dateDict[currentDate.hour][0] += 1
dateDict[currentDate.hour][1] += int(line[7])
```

For the entire dataset, the hour trend is shown.  A trend in the number of passengers in observed from the Midnight-4am hour, aligning with typical night establishment hours.  The passengers per trip takes a sharp drop, then slowly increases through the day.  Small jumps at 3pm and 6pm which are typical end of workday timeframes.
![passengers by hour, entire dataset](/fullPassengerPerHour.png)

The same plot was generated on the summarized dataset.  The sampled dataset shows more variation than the longer dataset.  Similar trends are noted with more granularity shown.  2am and 4am are standing out as well as 4-7pm.
![passengers by hour, smapled dataset](/samplePassengerPerHour.png)