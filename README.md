# Twitter Customer Support Collector

Collects tweets from companies providing customer support on Twitter.

Compatibility: Python 3.6 and up.

## Setup

The project uses Postgres (version 11), so you'll need to create a database for it:

```bash
$ sudo pg_createcluster -p 5455 11 twcs
$ sudo systemctl daemon-reload
$ sudo systemctl restart postgresql@11-twcs
$ sudo -Hu postgres psql -p 5455 -c "create role $USER superuser login;" && sudo -Hu postgres psql -p 5455 -c "create database twitter_cs with owner $USER;"
$ psql -p 5455 twitter_cs -f create.sql
```

Check that you can access the database and the user was created successfully:
```
service postgresql status # Postgres status
psql -p 5455 twitter_cs  # Getting into the DB shell
twitter_cs=# \du  # Command to show users
```

You'll need to provide the Twitter handles to monitor, consumer and access keys and tokens.
This can be done by setting env variables, or by providing them in the .env file.

```bash
$ cp example.env .env
$ vim .env # and then insert your keys/secrets, change accounts to scrape
```

## Run

Running the script:

```bash
$ PYTHONPATH=$(pwd) python3.6 main.py
```

You should be able to run it every 15 minutes without going over API limits, so running it from Jenkins or cron is a great match.

## Running cron job

Added to `crontab -e` file:

This will run the script every 15 minutes → rate limited by twitter or else:
```
*/15 * * * * cd ~/twitter-customer-support-collector && /home/paperspace/twitter-customer-support-collector/venv/bin/python /home/paperspace/twitter-customer-support-collector/main.py
```
Logs are written out to the `logs` directory.

To check the status of the cronjob:
```
service cron status
cat /var/log/syslog | grep twitter
```

## Checking the database
```
service postgresql status
psql -p 5455 twitter_cs
select count(*) from tweets;
SELECT pg_size_pretty( pg_database_size('twitter_cs') );
```

## Export

To export the dataset to a CSV, run:
```shell script
OUTFILE="exported.csv" python export.py
```
