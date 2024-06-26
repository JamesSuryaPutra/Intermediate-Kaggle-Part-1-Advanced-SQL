# JOINs & UNIONs example
We'll work with the Hacker News dataset. We begin by reviewing the first several rows of the comments table. 

    from google.cloud import bigquery

    # Create a "Client" object
    client = bigquery.Client()

    # Construct a reference to the "hacker_news" dataset
    dataset_ref = client.dataset("hacker_news", project="bigquery-public-data")

    # API request - fetch the dataset
    dataset = client.get_dataset(dataset_ref)

    # Construct a reference to the "comments" table
    table_ref = dataset_ref.table("comments")

    # API request - fetch the table
    table = client.get_table(table_ref)

    # Preview the first five lines of the table
    client.list_rows(table, max_results=5).to_dataframe()

    Using Kaggle's public dataset BigQuery integration.

    /opt/conda/lib/python3.7/site-packages/ipykernel_launcher.py:19: UserWarning:
    Cannot use bqstorage_client if max_results is set, reverting to fetching data with the tabledata.list endpoint.

	id	by	author	time	time_ts	text	parent	deleted	dead	ranking
    0	9734136	None	None	1434565400	2015-06-17 18:23:20+00:00	None	9733698	True	None	0
    1	4921158	None	None	1355496966	2012-12-14 14:56:06+00:00	None	4921100	True	None	0
    2	7500568	None	None	1396261158	2014-03-31 10:19:18+00:00	None	7499385	True	None	0
    3	8909635	None	None	1421627275	2015-01-19 00:27:55+00:00	None	8901135	True	None	0
    4	9256463	None	None	1427204705	2015-03-24 13:45:05+00:00	None	9256346	True	None	0


    # Construct a reference to the "stories" table
    table_ref = dataset_ref.table("stories")

    # API request - fetch the table
    table = client.get_table(table_ref)

    # Preview the first five lines of the table
    client.list_rows(table, max_results=5).to_dataframe()

    /opt/conda/lib/python3.7/site-packages/ipykernel_launcher.py:8: UserWarning:
    Cannot use bqstorage_client if max_results is set, reverting to fetching data with the tabledata.list endpoint.

    id	by	score	time	time_ts	title	url	text	deleted	dead	descendants	author
    0	6988445	cflick	0	1388454902	2013-12-31 01:55:02+00:00	Appshare	http://chadflick.ws/appshare.html	Did facebook or angrybirds pay you? We will!	None	True	NaN	cflick
    1	7047571	Rd2	1	1389562985	2014-01-12 21:43:05+00:00	Java in startups		Hello, hacker news!<p>Have any of you used jav...	None	True	NaN	Rd2
    2	9157712	mo0	1	1425657937	2015-03-06 16:05:37+00:00	Show HN: Discover what songs were used in YouT...	http://www.mooma.sh/	The user can paste a media url(currently only ...	None	True	NaN	mo0
    3	8127403	ad11	1	1407052667	2014-08-03 07:57:47+00:00	My poker project, what do you think?		Hi guys, what do you think about my poker proj...	None	True	NaN	ad11
    4	6933158	emyy	1	1387432701	2013-12-19 05:58:21+00:00	Christmas Crafts Ideas - Easy and Simple Famil...	http://www.winxdvd.com/resource/christmas-craf...	There are some free Christmas craft ideas to m...	None	True	NaN	emyy


Since you are already familiar with JOINs from the Intro to SQL micro-course, we'll work with a relatively complex example of a JOIN that uses a common table expression (CTE).
The query below pulls information from the stories and comments tables to create a table showing all stories posted on January 1, 2012, along with the corresponding number of comments.
We use a LEFT JOIN so that the results include stories that didn't receive any comments.

    # Query to select all stories posted on January 1, 2012, with number of comments
    join_query = """
                 WITH c AS
                 (
                  SELECT parent, COUNT(*) as num_comments
                  FROM `bigquery-public-data.hacker_news.comments` 
                  GROUP BY parent
                 )
                 SELECT s.id as story_id, s.by, s.title, c.num_comments
                 FROM `bigquery-public-data.hacker_news.stories` AS s
                 LEFT JOIN c
                 ON s.id = c.parent
                 WHERE EXTRACT(DATE FROM s.time_ts) = '2012-01-01'
                 ORDER BY c.num_comments DESC
                 """

    # Run the query, and return a pandas DataFrame
    join_result = client.query(join_query).result().to_dataframe()
    join_result.head()

    /opt/conda/lib/python3.7/site-packages/google/cloud/bigquery/client.py:440: UserWarning:
    Cannot create BigQuery Storage client, the dependency google-cloud-bigquery-storage is not installed.
      "Cannot create BigQuery Storage client, the dependency "

        story_id  by	        title	                                                num_comments
    0	3412900	  whoishiring	Ask HN: Who is Hiring? (January 2012)	                154.0
    1	3412901	  whoishiring	Ask HN: Freelancer? Seeking freelancer? (Janua...	97.0
    2	3412643	  jemeshsu	Avoid Apress	                                        30.0
    3	3414012	  ramanujam	Impress.js - a Prezi like implementation using...	27.0
    4	3412891	  Brajeshwar	There's no shame in code that is simply "good ...	27.0


Since the results are ordered by the num_comments column, stories without comments appear at the end of the DataFrame. (Remember that NaN stands for "not a number".)

    # None of these stories received any comments
    join_result.tail()

        story_id by	        title	                                                num_comments
    439	3413041	 ORioN63	Solar days, sidereal days, solar years and sid...	NaN
    440	3412667	 Tez_Dhar	How shall i Learn Hacking	                        NaN
    441	3412783	 mmaltiar	Working With Spring Data JPA	                        NaN
    442	3412821	 progga	        Networking on the Network: A Guide to Professi...	NaN
    443	3412930	 shipcode	Project Zero Operating System – New Kernel	        NaN


Next, we write a query to select all usernames corresponding to users who wrote stories or comments on January 1, 2014. We use UNION DISTINCT (instead of UNION ALL) to ensure that each
user appears in the table at most once.

    # Query to select all users who posted stories or comments on January 1, 2014
    union_query = """
                  SELECT c.by
                  FROM `bigquery-public-data.hacker_news.comments` AS c
                  WHERE EXTRACT(DATE FROM c.time_ts) = '2014-01-01'
                  UNION DISTINCT
                  SELECT s.by
                  FROM `bigquery-public-data.hacker_news.stories` AS s
                  WHERE EXTRACT(DATE FROM s.time_ts) = '2014-01-01'
                  """

    # Run the query, and return a pandas DataFrame
    union_result = client.query(union_query).result().to_dataframe()
    union_result.head()

    /opt/conda/lib/python3.7/site-packages/google/cloud/bigquery/client.py:440: UserWarning:
    Cannot create BigQuery Storage client, the dependency google-cloud-bigquery-storage is not installed.
      "Cannot create BigQuery Storage client, the dependency "

        by
    0	learnlivegrow
    1	egybreak
    2	dclara
    3	vram22
    4	espeed


To get the number of users who posted on January 1, 2014, we need only take the length of the DataFrame.

    # Number of users who posted stories or comments on January 1, 2014
    len(union_result)

    2282

