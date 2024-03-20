# README

This is our project about network performance analysis. The source code are in the `notebooks` folder. And `data` folder has the dataset we use, either from direct download from databases, SQL queries in Google BigQuery or xml fetch from webpage directly.

Since we have too many images through our work, we do not create a folder specifically for all of them. For interactive slider images, please refer to `graphs` folder.

Additionally, we provide SQL code we used to fetch the network dataset below. The link is here: https://console.cloud.google.com/bigquery?project=measurement-lab

random 5000 geolocation sample from 2011 to 2018:
```
    SELECT
    client.Geo.Latitude AS client_lat,
    client.Geo.Longitude AS client_lon,
    server.Geo.Latitude AS server_lat,
    server.Geo.Longitude AS server_lon,
    FROM
    `measurement-lab.ndt.web100`
    WHERE
    date BETWEEN '2011-01-01' AND '2018-12-31'
    AND client.Geo.CountryName = 'United States'
    AND client.Geo.Latitude IS NOT NULL
    AND client.Geo.Longitude IS NOT NULL
    AND server.Geo.Latitude IS NOT NULL
    AND server.Geo.Longitude IS NOT NULL
    AND a.MinRTT IS NOT NULL
    AND a.LossRate IS NOT NULL
    AND a.MeanThroughputMbps > 1
    AND a.MeanThroughputMbps < 100
    AND a.MinRTT < 300
    AND raw.connection.client_browser IS NOT NULL
    AND raw.connection.data_direction = 1
    AND raw.connection.client_ip IS NOT NULL
    AND raw.connection.server_ip IS NOT NULL
    AND raw.web100.connection_spec.local_ip IS NOT NULL
    AND raw.web100.connection_spec.remote_ip IS NOT NULL
    AND raw.web100.snap.CountRTT > 10
    AND raw.web100.snap.HCThruOctetsAcked >= 8192
    AND (raw.web100.snap.SndLimTimeRwin +
        raw.web100.snap.SndLimTimeCwnd +
        raw.web100.snap.SndLimTimeSnd) >= 5000000
    AND (raw.web100.snap.SndLimTimeRwin +
        raw.web100.snap.SndLimTimeCwnd +
        raw.web100.snap.SndLimTimeSnd) < 600000000
    AND (raw.web100.snap.State = 1
        OR (raw.web100.snap.State >= 5
        AND raw.web100.snap.State <= 11))
    ORDER BY RAND()
    LIMIT 5000
```

average throughput and average rtt by month from 2009 to 2018:
```
    SELECT
    EXTRACT(YEAR FROM date) AS year,
    EXTRACT(MONTH FROM date) AS month,
    AVG(a.MeanThroughputMbps) AS avg_throughput,
    AVG(a.MinRTT) as avg_rtt
    FROM
    `measurement-lab.ndt.web100`
    WHERE
    date BETWEEN '2009-01-01' AND '2018-12-31'
    AND client.Geo.CountryName = 'United States'
    AND client.Geo.Latitude IS NOT NULL
    AND client.Geo.Longitude IS NOT NULL
    AND server.Geo.Latitude IS NOT NULL
    AND server.Geo.Longitude IS NOT NULL
    AND a.MinRTT IS NOT NULL
    AND a.LossRate IS NOT NULL
    AND a.MeanThroughputMbps > 1
    AND a.MeanThroughputMbps < 100
    AND a.MinRTT < 300
    AND raw.connection.client_browser IS NOT NULL
    AND raw.connection.data_direction = 1
    AND raw.connection.client_ip IS NOT NULL
    AND raw.connection.server_ip IS NOT NULL
    AND raw.web100.connection_spec.local_ip IS NOT NULL
    AND raw.web100.connection_spec.remote_ip IS NOT NULL
    AND raw.web100.snap.CountRTT > 1
    AND raw.web100.snap.HCThruOctetsAcked >= 8192
    AND (raw.web100.snap.SndLimTimeRwin +
        raw.web100.snap.SndLimTimeCwnd +
        raw.web100.snap.SndLimTimeSnd) >= 5000000
    AND (raw.web100.snap.SndLimTimeRwin +
        raw.web100.snap.SndLimTimeCwnd +
        raw.web100.snap.SndLimTimeSnd) < 600000000
    AND (raw.web100.snap.State = 1
        OR (raw.web100.snap.State >= 5
        AND raw.web100.snap.State <= 11))
    GROUP BY year, month
    ORDER BY year, month
```

zip code and throughput pair for each measurement test only in california, we set up some filter requirements.
If you want to fully reproduce the process, use the date format as '20xx-01-01' and '20xx-06-30' with '20xx-07-01' and '20xx-12-31'.
```
    SELECT
    client.Geo.PostalCode AS zip,
    a.MeanThroughputMbps AS throughput
    FROM
    `measurement-lab.ndt.ndt7`
    WHERE
    date BETWEEN '2021-01-01' AND '2021-06-30'
    AND client.Geo.CountryName = 'United States'
    AND client.Geo.Latitude IS NOT NULL
    AND client.Geo.Longitude IS NOT NULL
    AND server.Geo.Latitude IS NOT NULL
    AND server.Geo.Longitude IS NOT NULL
    AND client.Geo.PostalCode IS NOT NULL
    AND a.MeanThroughputMbps > 20
    AND a.MeanThroughputMbps < 200
    AND client.Geo.PostalCode BETWEEN '90001' AND '96162'
    AND SQRT(
        POW(server.Geo.Latitude - client.Geo.Latitude, 2) + POW(server.Geo.Longitude - client.Geo.Longitude, 2)
    ) < 10
    ORDER BY RAND()
    LIMIT 100000
```