# Troubleshooting N:N JOINs
Now we'll work with an example from a real dataset. Both examples below count the number of distinct committers and the number of files in several GitHub repositories.

    big_join_query = """
                     SELECT repo,
                     COUNT(DISTINCT c.committer.name) as num_committers,
                     COUNT(DISTINCT f.id) AS num_files
                     FROM `bigquery-public-data.github_repos.commits` AS c,
                     UNNEST(c.repo_name) AS repo
                     INNER JOIN `bigquery-public-data.github_repos.files` AS f
                     ON f.repo_name = repo
                     WHERE f.repo_name IN ( 'tensorflow/tensorflow', 'facebook/react', 'twbs/bootstrap', 'apple/swift', 'Microsoft/vscode', 'torvalds/linux')
                     GROUP BY repo
                     ORDER BY repo
                     """

    show_time_to_run(big_join_query)

    small_join_query = """
                       WITH commits AS
                       (
                        SELECT COUNT(DISTINCT committer.name) AS num_committers, repo
                        FROM `bigquery-public-data.github_repos.commits`,
                        UNNEST(repo_name) as repo
                        WHERE repo IN ( 'tensorflow/tensorflow', 'facebook/react', 'twbs/bootstrap', 'apple/swift', 'Microsoft/vscode', 'torvalds/linux')
                        GROUP BY repo
                       ),
                       files AS 
                       (
                        SELECT COUNT(DISTINCT id) AS num_files, repo_name as repo
                        FROM `bigquery-public-data.github_repos.files`
                        WHERE repo_name IN ( 'tensorflow/tensorflow', 'facebook/react', 'twbs/bootstrap', 'apple/swift', 'Microsoft/vscode', 'torvalds/linux')
                        GROUP BY repo
                       )
                       SELECT commits.repo, commits.num_committers, files.num_files
                       FROM commits 
                       INNER JOIN files
                       ON commits.repo = files.repo
                       ORDER BY repo
                       """

    show_time_to_run(small_join_query)

    Time to run: 13.028 seconds
    Time to run: 4.413 seconds


The first query has a large N:N JOIN. By rewriting the query to decrease the size of the JOIN, we see it runs much faster.
