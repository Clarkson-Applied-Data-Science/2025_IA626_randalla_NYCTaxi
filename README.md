test
##Date Range
The date range was determined using the trip pickup times.  Each row was converted to a DT object and compared against previous entries to determine the most recent and most historic values.  Default values in the past/future were used to initialize the values.  The dates could also have been evaluated based on the drop-off time to get some more recent items, though this effort was focused on the pickup times.

##Field Names
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

##Geographic Area
The geographic area plot is provided below.  A more indepth description of the filtering method is provided in [here](#filtering).   The area generally aligns with the tri-state area that would be typical of a NYC taxi trip.

![A plot of the coordinates for the large dataset](/GeographicArea.png)


##Filtering outliers and invalid data <a name ="filtering"></a>
Several iterations of filtering were attempted and an effort was made to generalize the solution.  Initially, a fixed bounding of lat/lon coordinates was used to force all trips to occur within a given geographic area.  This is somewhat difficult to apply more generally, since new unique coordinates would be required each time.

Another angle was to use the trip distance and compare it to the lat/lon havershine distances.  This removes items where the lat/lon values and the trip_distance values do not align.  Some of the trips included in this data set appear to occur outside of the geographic region or have invalid lat/lon values and caused outliers to persist through the filtering.  Some trips occured only within Tennessee and some others occured via boats in the middle of the Atlantic Ocean apparently.  Since this effort was focused around NYC Taxi trips, these can be ignored.

The end solution was to use a fixed geographic point and use a radius around it to filter trips that did not start within a reasonable distance.  In this case a 100km radius was used around central NYC coordinates gathered from wikipedia.  This solution allows more options in the future to apply a similar code structure to other geographic areas without requiring major code revisions.

![another one](/NYC_Radius.png)
