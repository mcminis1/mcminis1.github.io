---
layout: post
title:  "Fast Filtering IP addresses for Cloud provider(s) from Snowflake using Python"
date:   2023-05-07 00:00:00 -0500
categories: jekyll update
regenerate: true
---

Let's say you want to see what traffic you have coming form Azure, GCP, or AWS over the last 6 months using Python.

Here's a snippet for you!

## Azure

```python
import json
from netaddr import IPSet, IPAddress
import snowflake.connector

# download from https://www.microsoft.com/en-us/download/confirmation.aspx?id=56519
az_net = json.load(open('ServiceTags_Public_20230501.json'))

az_ips = []
for az in az_net['values']:
    az_ips += az['properties']['addressPrefixes']
az_ip_set = IPSet(az_ips)


con = snowflake.connector.connect(
    user='user',
    password='password',
    account='account',
    role="role",
    database="database",
    warehouse="warehouse"
)

print("month, n_azure_ips, n_new_ips, pct_azure")

all_matches = []
months = ['2023-05-01','2023-04-01','2023-03-01','2023-02-01','2023-01-01','2022-12-01','2022-11-01','2022-10-01']
for month in months:
    query = f"select distinct ip from DATABASE.SCHEMA.TABLE where DATE_TRUNC('month', CREATED_ON) = '{month}'"

    ip_list = con.cursor().execute(query).fetchall()
    matches = []
    for ip_addr in ip_list:
        if IPAddress(ip_addr[0]) in az_ip_set:
            matches.append(ip_addr[0])

    n_ips = len(ip_list)
    n_az_ips = len(matches)
    all_matches += matches

    print(f"{month}, {n_az_ips}, {n_ips}, {100.0*n_az_ips/n_ips}")
```

Note that if you want to upload these back to snowflake, you'll need to batch them in groups of 16K or less.

## AWS

All you have to do for the other providers ios change out the ip address blocks.

```python
# https://ip-ranges.amazonaws.com/ip-ranges.json
az_net = json.load(open('aws-ip-ranges.json'))

az_ips = []
for az in az_net['prefixes']:
    az_ips.append(az['ip_prefix'])
az_ip_set = IPSet(az_ips)
```

## GCP

```python
# https://www.gstatic.com/ipranges/cloud.json
az_net = json.load(open('gcp-cloud-ip-ranges.json'))

az_ips = []
for az in az_net['prefixes']:
    if "ipv4Prefix" in az:
        az_ips.append(az['ipv4Prefix'])
    else:
        az_ips.append(az['ipv6Prefix'])

# https://www.gstatic.com/ipranges/goog.json
az_net = json.load(open('gcp-goog-ip-ranges.json'))
for az in az_net['prefixes']:
    if "ipv4Prefix" in az:
        az_ips.append(az['ipv4Prefix'])
    else:
        az_ips.append(az['ipv6Prefix'])
az_ip_set = IPSet(az_ips)
```