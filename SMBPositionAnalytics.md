SMB Position Analytics Service
==============================

#### Table of Contents
_______

- [Summary](#summary)
- [Requests](#requests)
    - [Carotene With Location Request](#carotene-with-location-request)
    - [Job Request](#job-request)
- [Responses](#responses)
    - [Carotene Response Structure](#carotene-response-structure)
    - [Job Response Structure](#job-response)
- [Versioning](#versioning)

## Summary

The Small Medium Business (SMB) Position Analytics service provides hiring data pertaining to job titles and currently open positions available on CareerBuilder job boards. The service has two endpoints:

https://api.careerbuilder.com/core/positionanalytics/carotene

https://api.careerbuilder.com/core/positionanalytics/job

Requests can be sent in the form of HTTP GET or POST (form or JSON)

 This service requires CareerBuilder OAuth 2.0 client credentials. For more information please see the authorization [documentation](/Readme.md#access).

## Requests
### Carotene With Location Request
A request to the carotene endpoint provides data on how a specified job title performs in a given location.

E.g. I'm a recruiter and I'm about to post a Software Engineer job in Atlanta. How many applies have there been to Software Engineer titles in this city in the last day? What is the change from the day before? On average how long does it take to fill a Software Engineer role in this location? This endpoint serves to answer these questions.

Requests consist of:

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `carotene_id` | string | true | Unique carotene identifier v3.1.
| `city` | string | true | Full name of a city respective of the carotene title
| `state` | string | true |Two letter abbreviation for the state

Example cURL request:

```
curl -X POST \
  https://api.careerbuilder.com/core/positionanalytics/carotene \
  -H 'Accept: application/json;version=1.0' \
  -H 'Authorization: <BEARER_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{
	"carotene_id": "11.0",
	"city": "Atlanta",
	"state": "GA",
      }'
```

Currently only Carotene v3.1 is supported. See the documentation for [Job Title service](https://github.com/careerbuilder/DataScienceAPIDocumentation/blob/master/JobTitle.md)
for more information about carotene classification and carotene IDs.

### Job Request
A request to the job endpoint requires a CareerBuilder Job Identifier also known as a Job DID and serves data pertaining to active jobs (open positions). Relevant data points returned include: number of clicks, number of applies, number of views, and total number of actions.

Requests consist of:

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `job_did` | string | true | Unique CareerBuilder job identifier.


Example cURL request:

```
curl -X POST \
  https://api.careerbuilder.com/core/positionanalytics/job \
  -H 'Accept: application/json;version=1.0' \
  -H 'Authorization: <BEARER_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{
	"job_did": "J2N21P76BM530W9HRQG",
      }'
```

## Responses
The data returned in the response will contain an object or group of objects representing actions taken on a carotene title or job respectively. An action is an operation that has been performed on a job, this can be a mouse click, an apply, or a view. Each action will have associated `current` and `delta` values. The `current` value will be the total number of times an action was performed over the last day; the `delta` will identify the change in percentage of the `current` value from the day before. Positive values in the `delta` identify an increase day over day and negative values identify a decrease.

 **Delta calculation formula:** *((current - previous)/previous)\*100 = delta*

 Each response will return a trailing `timestamp` string. This is the date time in which the data was last refreshed. The `timestamp` string is returned in ISO 8601 format with UTC offset(yyyy-MM-dd'T'HH:mm:ss-XXX). Clients can expect that data will be refreshed daily to account for the changes to the `current` and  `delta` values which outline actions performed in the last day.

### Carotene Response Structure

 `num_of_applies` holds the `current` number of applies within the last day period and `delta` percent change in applies from the previous day.

 `avg_time_to_fill` is the time period in days on average to fill a position identified by the `carotene_id`.


```json
{
    "data": {
        "num_of_applies": {
            "current": 2,
            "delta": -1.39393
        },
        "avg_time_to_fill": 23,
        "timestamp": "2019-01-04T10:00:32-05:00"
    }
}
```

### Job Response Structure
The actions returned with a job request include: `num_of_clicks`, `num_of_applies`, `num_of_views`, and `total_num_actions`. Each action contains an associated `current` and `delta` value.

```json
{
  "data": {
    "num_of_clicks": {
      "current": 8,
      "delta": -72.41379310344827
    },
    "num_of_applies": {
      "current": 1,
      "delta": -50.0
    },
    "num_of_views": {
      "current": 2,
      "delta": -83.33333333333333
    },
    "total_num_actions": {
      "current": 9,
      "delta": -70.96774193548387
    },
    "timestamp": "2019-01-04T10:00:32-05:00"
  }
}
```


#### No Results found:
**When no data is found for the request, the service will return an empty data object.**

```json
{
    "data": {
    }
}
```

## Versioning
The current version of the service is 1.0.

Version must be specified in the Accept header. E.g. `application/json;version=1.0`.

Our general versioning strategy is available [here](/Versioning.md).