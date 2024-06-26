# Nested & repeated data
To reinforce what you've learned, we'll apply these ideas to a real dataset in the section below.

We'll work with the Google Analytics Sample dataset. It contains information tracking the behavior of visitors to the Google Merchandise store, an e-commerce website that sells Google
branded items. We begin by printing the first few rows of the ga_sessions_20170801 table.

    from google.cloud import bigquery

    # Create a "Client" object
    client = bigquery.Client()

    # Construct a reference to the "google_analytics_sample" dataset
    dataset_ref = client.dataset("google_analytics_sample", project="bigquery-public-data")

    # Construct a reference to the "ga_sessions_20170801" table
    table_ref = dataset_ref.table("ga_sessions_20170801")

    # API request - fetch the table
    table = client.get_table(table_ref)

    # Preview the first five lines of the table
    client.list_rows(table, max_results=5).to_dataframe()

    Using Kaggle's public dataset BigQuery integration.

    /opt/conda/lib/python3.7/site-packages/ipykernel_launcher.py:16: UserWarning:
    Cannot use bqstorage_client if max_results is set, reverting to fetching data with the tabledata.list endpoint.

    visitorId	visitNumber	visitId	visitStartTime	date	totals	trafficSource	device	geoNetwork	customDimensions	hits	fullVisitorId	userId	clientId	channelGrouping	socialEngagementType
    0	NaN	1	1501591568	1501591568	20170801	{'visits': 1, 'hits': 1, 'pageviews': 1, 'time...	{'referralPath': None, 'campaign': '(not set)'...	{'browser': 'Chrome', 'browserVersion': 'not a...	{'continent': 'Europe', 'subContinent': 'South...	[]	[{'hitNumber': 1, 'time': 0, 'hour': 5, 'minut...	3418334011779872055	None	None	Organic Search	Not Socially Engaged
    1	NaN	2	1501589647	1501589647	20170801	{'visits': 1, 'hits': 1, 'pageviews': 1, 'time...	{'referralPath': '/analytics/web/', 'campaign'...	{'browser': 'Chrome', 'browserVersion': 'not a...	{'continent': 'Asia', 'subContinent': 'Souther...	[{'index': 4, 'value': 'APAC'}]	[{'hitNumber': 1, 'time': 0, 'hour': 5, 'minut...	2474397855041322408	None	None	Referral	Not Socially Engaged
    2	NaN	1	1501616621	1501616621	20170801	{'visits': 1, 'hits': 1, 'pageviews': 1, 'time...	{'referralPath': '/analytics/web/', 'campaign'...	{'browser': 'Chrome', 'browserVersion': 'not a...	{'continent': 'Europe', 'subContinent': 'North...	[{'index': 4, 'value': 'EMEA'}]	[{'hitNumber': 1, 'time': 0, 'hour': 12, 'minu...	5870462820713110108	None	None	Referral	Not Socially Engaged
    3	NaN	1	1501601200	1501601200	20170801	{'visits': 1, 'hits': 1, 'pageviews': 1, 'time...	{'referralPath': '/analytics/web/', 'campaign'...	{'browser': 'Firefox', 'browserVersion': 'not ...	{'continent': 'Americas', 'subContinent': 'Nor...	[{'index': 4, 'value': 'North America'}]	[{'hitNumber': 1, 'time': 0, 'hour': 8, 'minut...	9397809171349480379	None	None	Referral	Not Socially Engaged
    4	NaN	1	1501615525	1501615525	20170801	{'visits': 1, 'hits': 1, 'pageviews': 1, 'time...	{'referralPath': '/analytics/web/', 'campaign'...	{'browser': 'Chrome', 'browserVersion': 'not a...	{'continent': 'Americas', 'subContinent': 'Nor...	[{'index': 4, 'value': 'North America'}]	[{'hitNumber': 1, 'time': 0, 'hour': 12, 'minu...	6089902943184578335	None	None	Referral	Not Socially Engaged


The table has many nested fields. In our first query against this table, we'll work with the "totals" and "device" columns.

    print("SCHEMA field for the 'totals' column:\n")
    print(table.schema[5])

    print("\nSCHEMA field for the 'device' column:\n")
    print(table.schema[7])

    SCHEMA field for the 'totals' column:
    SchemaField('totals', 'RECORD', 'NULLABLE', None, (SchemaField('visits', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('hits', 'INTEGER', 'NULLABLE', None, (), None),
    SchemaField('pageviews', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('timeOnSite', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('bounces', 'INTEGER', 'NULLABLE', None, (),
    None), SchemaField('transactions', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('transactionRevenue', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('newVisits', 'INTEGER',
    'NULLABLE', None, (), None), SchemaField('screenviews', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('uniqueScreenviews', 'INTEGER', 'NULLABLE', None, (), None),
    SchemaField('timeOnScreen', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('totalTransactionRevenue', 'INTEGER', 'NULLABLE', None, (), None), SchemaField('sessionQualityDim',
    'INTEGER', 'NULLABLE', None, (), None)), None)

    SCHEMA field for the 'device' column:
    SchemaField('device', 'RECORD', 'NULLABLE', None, (SchemaField('browser', 'STRING', 'NULLABLE', None, (), None), SchemaField('browserVersion', 'STRING', 'NULLABLE', None, (), None),
    SchemaField('browserSize', 'STRING', 'NULLABLE', None, (), None), SchemaField('operatingSystem', 'STRING', 'NULLABLE', None, (), None), SchemaField('operatingSystemVersion', 'STRING',
    'NULLABLE', None, (), None), SchemaField('isMobile', 'BOOLEAN', 'NULLABLE', None, (), None), SchemaField('mobileDeviceBranding', 'STRING', 'NULLABLE', None, (), None),
    SchemaField('mobileDeviceModel', 'STRING', 'NULLABLE', None, (), None), SchemaField('mobileInputSelector', 'STRING', 'NULLABLE', None, (), None), SchemaField('mobileDeviceInfo',
    'STRING', 'NULLABLE', None, (), None), SchemaField('mobileDeviceMarketingName', 'STRING', 'NULLABLE', None, (), None), SchemaField('flashVersion', 'STRING', 'NULLABLE', None, (), None),
    SchemaField('javaEnabled', 'BOOLEAN', 'NULLABLE', None, (), None), SchemaField('language', 'STRING', 'NULLABLE', None, (), None), SchemaField('screenColors', 'STRING', 'NULLABLE', None,
    (), None), SchemaField('screenResolution', 'STRING', 'NULLABLE', None, (), None), SchemaField('deviceCategory', 'STRING', 'NULLABLE', None, (), None)), None)


We refer to the "browser" field (which is nested in the "device" column) and the "transactions" field (which is nested inside the "totals" column) as device.browser and
totals.transactions in the query below.

    # Query to count the number of transactions per browser
    query = """
            SELECT device.browser AS device_browser,
            SUM(totals.transactions) as total_transactions
            FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
            GROUP BY device_browser
            ORDER BY total_transactions DESC
            """

    # Run the query, and return a pandas DataFrame
    result = client.query(query).result().to_dataframe()
    result.head()

    /opt/conda/lib/python3.7/site-packages/google/cloud/bigquery/client.py:440: UserWarning:
    Cannot create BigQuery Storage client, the dependency google-cloud-bigquery-storage is not installed.
      "Cannot create BigQuery Storage client, the dependency "

        device_browser	total_transactions
    0	Chrome	        41.0
    1	Safari	        3.0
    2	Firefox	        1.0
    3	Edge	        NaN
    4	Coc Coc	        NaN


By storing the information in the "device" and "totals" columns as STRUCTs (as opposed to separate tables), we avoid expensive JOINs. This increases performance and keeps us from having
to worry about JOIN keys (and which tables have the exact data we need).

Now we'll work with the "hits" column as an example of data that is both nested and repeated. Since:

1} "hits" is a STRUCT (contains nested data) and is repeated,

2} "hitNumber", "page", and "type" are all nested inside the "hits" column, and

3} "pagePath" is nested inside the "page" field,

we can query these fields with the following syntax.

    # Query to determine most popular landing point on the website
    query = """
            SELECT hits.page.pagePath as path,
            COUNT(hits.page.pagePath) as counts
            FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`, 
            UNNEST(hits) as hits
            WHERE hits.type="PAGE" and hits.hitNumber=1
            GROUP BY path
            ORDER BY counts DESC
            """

    # Run the query, and return a pandas DataFrame
    result = client.query(query).result().to_dataframe()
    result.head()

    /opt/conda/lib/python3.7/site-packages/google/cloud/bigquery/client.py:440: UserWarning:
    Cannot create BigQuery Storage client, the dependency google-cloud-bigquery-storage is not installed.
      "Cannot create BigQuery Storage client, the dependency "

        path	                                        counts
    0	/home	                                        1257
    1	/google+redesign/shop+by+brand/youtube	        587
    2	/google+redesign/apparel/mens/mens+t+shirts	117
    3	/signin.html	                                78
    4	/basket.html	                                35


In this case, most users land on the website through the "/home" page.
