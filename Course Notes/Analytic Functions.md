# Analytic functions example
We'll work with the San Francisco Open Data dataset. We begin by reviewing the first several rows of the bikeshare_trips table. 

    from google.cloud import bigquery

    # Create a "Client" object
    client = bigquery.Client()

    # Construct a reference to the "san_francisco" dataset
    dataset_ref = client.dataset("san_francisco", project="bigquery-public-data")

    # API request - fetch the dataset
    dataset = client.get_dataset(dataset_ref)

    # Construct a reference to the "bikeshare_trips" table
    table_ref = dataset_ref.table("bikeshare_trips")

    # API request - fetch the table
    table = client.get_table(table_ref)

    # Preview the first five lines of the table
    client.list_rows(table, max_results=5).to_dataframe()

    Using Kaggle's public dataset BigQuery integration.

    /opt/conda/lib/python3.7/site-packages/ipykernel_launcher.py:19: UserWarning:
    Cannot use bqstorage_client if max_results is set, reverting to fetching data with the tabledata.list endpoint.

    trip_id	duration_sec	start_date	start_station_name	start_station_id	end_date	end_station_name	end_station_id	bike_number	zip_code	subscriber_type
    0	1235850	1540	2016-06-11 08:19:00+00:00	San Jose Diridon Caltrain Station	2	2016-06-11 08:45:00+00:00	San Jose Diridon Caltrain Station	2	124	15206	Customer
    1	1219337	6324	2016-05-29 12:49:00+00:00	San Jose Diridon Caltrain Station	2	2016-05-29 14:34:00+00:00	San Jose Diridon Caltrain Station	2	174	55416	Customer
    2	793762	115572	2015-06-04 09:22:00+00:00	San Jose Diridon Caltrain Station	2	2015-06-05 17:28:00+00:00	San Jose Diridon Caltrain Station	2	190	95391	Customer
    3	453845	54120	2014-09-15 16:53:00+00:00	San Jose Diridon Caltrain Station	2	2014-09-16 07:55:00+00:00	San Jose Diridon Caltrain Station	2	127	81	Customer
    4	1245113	5018	2016-06-17 20:08:00+00:00	San Jose Diridon Caltrain Station	2	2016-06-17 21:32:00+00:00	San Jose Diridon Caltrain Station	2	153	95070	Customer


Each row of the table corresponds to a different bike trip, and we can use an analytic function to calculate the cumulative number of trips for each date in 2015.

    # Query to count the (cumulative) number of trips per day
    num_trips_query = """
                      WITH trips_by_day AS
                      (
                       SELECT DATE(start_date) AS trip_date,
                       COUNT(*) as num_trips
                       FROM `bigquery-public-data.san_francisco.bikeshare_trips`
                       WHERE EXTRACT(YEAR FROM start_date) = 2015
                       GROUP BY trip_date
                      )
                      SELECT *,
                      SUM(num_trips) 
                          OVER (
                                ORDER BY trip_date
                                ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
                               ) AS cumulative_trips
                     FROM trips_by_day
                     """

    # Run the query, and return a pandas DataFrame
    num_trips_result = client.query(num_trips_query).result().to_dataframe()
    num_trips_result.head()

    /opt/conda/lib/python3.7/site-packages/google/cloud/bigquery/client.py:440: UserWarning:
    Cannot create BigQuery Storage client, the dependency google-cloud-bigquery-storage is not installed.
      "Cannot create BigQuery Storage client, the dependency "

    trip_date	num_trips	cumulative_trips
    0	2015-01-20	1213	16742
    1	2015-01-22	1224	19280
    2	2015-04-22	1376	108778
    3	2015-03-23	1247	76875
    4	2015-09-04	1129	248182


The query uses a common table expression (CTE) to first calculate the daily number of trips. Then, we use SUM() as an aggregate function.

1} Since there is no PARTITION BY clause, the entire table is treated as a single partition.

2} The ORDER BY clause orders the rows by date, where earlier dates appear first.

3} By setting the window frame clause to ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW, we ensure that all rows up to and including the current date are used to calculate the
cumulative sum (Note: If you read the documentation, you'll see that this is the default behavior, and so the query would return the same result if we left out this window frame
clause).

The next query tracks the stations where each bike began (in start_station_id) and ended (in end_station_id) the day on October 25, 2015.

    # Query to track beginning and ending stations on October 25, 2015, for each bike
    start_end_query = """
                      SELECT bike_number,
                      TIME(start_date) AS trip_time,
                      FIRST_VALUE(start_station_id)
                          OVER (
                               PARTITION BY bike_number
                               ORDER BY start_date
                               ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
                               ) AS first_station_id,
                      LAST_VALUE(end_station_id)
                          OVER (
                               PARTITION BY bike_number
                               ORDER BY start_date
                               ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
                               ) AS last_station_id,
                      start_station_id,
                      end_station_id
                      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
                      WHERE DATE(start_date) = '2015-10-25' 
                      """

    # Run the query, and return a pandas DataFrame
    start_end_result = client.query(start_end_query).result().to_dataframe()
    start_end_result.head()

    /opt/conda/lib/python3.7/site-packages/google/cloud/bigquery/client.py:440: UserWarning:
    Cannot create BigQuery Storage client, the dependency google-cloud-bigquery-storage is not installed.
      "Cannot create BigQuery Storage client, the dependency "

    bike_number	trip_time	first_station_id	last_station_id	start_station_id	end_station_id
    0	294	14:29:00	64	77	64	77
    1	395	00:24:00	70	72	70	72
    2	480	15:56:00	39	67	39	67
    3	581	12:41:00	77	73	77	60
    4	581	12:56:00	77	73	60	73


The query uses both FIRST_VALUE() and LAST_VALUE() as analytic functions.

1} The PARTITION BY clause breaks the data into partitions based on the bike_number column. Since this column holds unique identifiers for the bikes, this ensures the calculations are
performed separately for each bike.

2} The ORDER BY clause puts the rows within each partition in chronological order.

3} Since the window frame clause is ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING, for each row, its entire partition is used to perform the calculation (This ensures the
calculated values for rows in the same partition are identical).
