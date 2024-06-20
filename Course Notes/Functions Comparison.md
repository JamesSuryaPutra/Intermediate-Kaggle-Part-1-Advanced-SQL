# Comparison of generating BigQuery functions
We will use two functions to compare the efficiency of different queries:

1} show_amount_of_data_scanned() shows the amount of data the query uses.

2} show_time_to_run() prints how long it takes for the query to execute.

    from google.cloud import bigquery
    from time import time

    client = bigquery.Client()

    def show_amount_of_data_scanned(query):
        # dry_run lets us see how much data the query uses without running it
        dry_run_config = bigquery.QueryJobConfig(dry_run=True)
        query_job = client.query(query, job_config=dry_run_config)
        print('Data processed: {} GB'.format(round(query_job.total_bytes_processed / 10**9, 3)))
    
    def show_time_to_run(query):
        time_config = bigquery.QueryJobConfig(use_query_cache=False)
        start = time()
        query_result = client.query(query, job_config=time_config).result()
        end = time()
        print('Time to run: {} seconds'.format(round(end-start, 3)))

    Using Kaggle's public dataset BigQuery integration.

