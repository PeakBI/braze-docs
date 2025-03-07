---
nav_title: "POST: User Profile Export by Segment"
article_title: "POST: User Profile Export by Segment"
search_tag: Endpoint
page_order: 4
layout: api_page
page_type: reference
description: "This article outlines details about the Users by Segment Braze endpoint."

---
{% api %}
# Users by segment endpoint
{% apimethod post %}
/users/export/segment
{% endapimethod %}

This endpoint allows you to export all the users within a segment. User data is exported as multiple files of user JSON objects separated by new lines (i.e., one JSON object per line).

Note that a company may run at most one export per segment using this endpoint at a given time. Wait for your export to complete before retrying. 

This endpoint is currently not supported by Google Cloud Storage.

{% apiref postman %}https://documenter.getpostman.com/view/4689407/SVYrsdsG?version=latest#cfa6fa98-632c-4f25-8789-6c3f220b9457 {% endapiref %}

{% alert important %}
Beginning December 2021, the following changed for this API:<br><br>1. The `fields_to_export` field in this API request is **required**. The option to default to all fields has been removed.<br>2. The fields for `custom_events`, `purchases`, `campaigns_received`, and `canvases_received` only contain data from the last 90 days.
{% endalert %}

## Rate limit

{% include rate_limits.md endpoint='default' %}

## Credentials-based response details

If you have added your S3 credentials to Braze, then each file will be uploaded in your bucket as a ZIP file with the key format that looks like `segment-export/SEGMENT_ID/YYYY-MM-dd/RANDOM_UUID-TIMESTAMP_WHEN_EXPORT_STARTED/filename.zip`. We will create 1 file per 5,000 users to optimize processing. You can then unzip the files and concatenate all of the `json` files to a single file if needed. If you specify an `output_format` of `gzip`, then the file extension will be `.gz` instead of `.zip`.

{% details Export Pathing Breakdown for ZIP File %}
ZIP file format:
`bucket-name/segment-export/SEGMENT_ID/YYYY-MM-dd/RANDOM_UUID-TIMESTAMP_WHEN_EXPORT_STARTED/filename.zip`

Example ZIP File:
`braze.docs.bucket/segment-export/abc56c0c-rd4a-pb0a-870pdf4db07q/2019-04-25/d9696570-dfb7-45ae-baa2-25e302r2da27-1556044807/114f0226319130e1a4770f2602b5639a.zip`

| Property | Details | Shown in Example as... |
|---|---|
| `bucket-name` | Fixed based on your bucket name. | `braze.docs.bucket`
| `segment-export` | Fixed. | `segment-export`
| `SEGMENT_ID` | Included in the export request. | `abc56c0c-rd4a-pb0a-870pdf4db07q`
| `YYYY-MM-dd` | The date the successful callback is received. | `2019-04-25`
| `RANDOM_UUID` | An out-of-the-box random UUID generated by Braze at the time of the request. | `d9696570-dfb7-45ae-baa2-25e302r2da27`
| `TIMESTAMP_WHEN_EXPORT_STARTED` | Unix time (seconds since 2017-01-01:00:00:00Z) that the export was requested. | `1556044807`
| `filename` | Random per file. | `114f0226319130e1a4770f2602b5639a`
{: .reset-td-br-1 .reset-td-br-2 .reset-td-br-3}

{% enddetails %}

If you do not have S3 credentials provided but have an Azure integration set up with Braze, the data can be exported there if you have the **Make this the default data export destination** box checked in the Azure partner overview page in Braze. If not, the response to the request provides an obfuscated URL where a zip file containing all the user files can be downloaded. The URL will only become a valid location once the export is ready. We strongly suggest that customers who use this endpoint set up their own S3 credentials so that customers can enforce their own S3 bucket policies on the export.

Be aware that if you do not have S3 credentials, there is a limitation on the amount of data that you can export from this endpoint. Depending on the fields you’re exporting and the number of users, the file transfer may fail if it is too large. A best practice is to specify which fields you want to export using ‘fields_to_export’ and specifying only the fields you need, in order to keep the size of the transfer lower. If you want to export all your users and are getting errors generating the file, consider breaking your user base up into more segments based on a random bucket number (e.g. create a segment where random bucket number <1000, between 1000 and 2000, etc).

In either scenario, you may optionally provide a `callback_endpoint` to be notified when the export is ready. If the `callback_endpoint` is provided, we will make a post request to the provided address when the download is ready. The body of the post will be "success":true. If you have not added S3 credentials to Braze, then the body of the post will additionally have the attribute `url` with the download URL as the value.

Larger user bases will result in longer export times. For example, an app with 20 million users could take an hour or more.

## Request body

```
Content-Type: application/json
Authorization: Bearer YOUR-REST-API-KEY
```

```json
{
    "segment_id" : (required, string) identifier for the segment to be exported,
    "callback_endpoint" : (optional, string) endpoint to post a download URL to when the export is available,
    "fields_to_export" : (required, array of string) name of user data fields to export, you may also export custom attributes. *Beginning April 2021, new accounts must specify specific fields to export.
    "output_format" : (optional, string) when using your own S3 bucket,  specifies file format as 'zip' or 'gzip'. Defaults to zip file format
}
```

The `segment_id` for a given segment can be found in your Developer Console within your Braze account or you can use the [Segment List Endpoint]({{site.baseurl}}/api/endpoints/export/get_segment/).

{% alert warning %}
Individual custom attributes cannot be exported. However, all custom attributes can be exported by including `custom_attributes` in the `fields_to_export` array (e.g., ['first_name', 'email', 'custom_attributes']).
{% endalert %}

## Request parameters

| Parameter | Required | Data Type | Description |
|---|---|---|---|
|`segment_id` | Required | String | Identifier for the segment to be exported. See [segment identifier]({{site.baseurl}}/api/identifier_types/). |
|`callback_endpoint` | Optional | String | Endpoint to post a download URL to when the export is available. |
|`fields_to_export` | Required* | Array of Strings | Name of user data fields to export, you may also export custom attributes. <br><br>*Beginning April 2021, new accounts must specify specific fields to export. |
|`output_format` | Optional | String | When using your own S3 bucket, allows to specify file format as `zip` or `gzip`. Defaults to ZIP file format. |
{: .reset-td-br-1 .reset-td-br-2 .reset-td-br-3  .reset-td-br-4}

## Example request
```
curl --location --request POST 'https://rest.iad-01.braze.com/users/export/segment' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR-REST-API-KEY' \
--data-raw '{
    "segment_id" : "segment_identifier",
    "callback_endpoint" : "example_endpoint",
    "fields_to_export" : ["first_name", "email", "purchases"],
    "output_format" : "zip"
}'
```

## Fields to export

The following is a list of valid `fields_to_export`. Using `fields_to_export` to minimize the data returned can improve response time of this API endpoint:

* `apps`
* `attributed_campaign`
* `attributed_source`
* `attributed_adgroup`
* `attributed_ad`
* `braze_id`
* `campaigns_received`
* `canvases_received`
* `cards_clicked`
* `country`
* `created_at`
* `custom_attributes`
* `custom_events`
* `devices`
* `dob`
* `email`
* `email_subscribe`
* `external_id`
* `first_name`
* `gender`
* `home_city`
* `language`
* `last_coordinates`
* `last_name`
* `phone`
* `purchases`
* `push_subscribe`
* `push_tokens`
* `random_bucket`
* `time_zone`
* `total_revenue`
* `uninstalled_at`
* `user_aliases`

## Important reminders

- The fields for `custom_events`, `purchases`, `campaigns_received`, and `canvases_received` will contain only contain data from the last 90 days.
- Both `custom_events` and `purchases` contain fields for `first` and `count`. Both of these fields will reflect information from all time, and will not be limited to just data from the last 90 days. For example, if a particular user first did the event prior to 90 days ago, this will be accurately reflected in the `first` field, and the `count` field will take into account events that occurred prior to the last 90 days as well.
- The number of concurrent segment exports a company can run at the endpoint level is capped at 100. Attempts that surpass this limit will result in an error.

## Response

```json
Content-Type: application/json
Authorization: Bearer YOUR-REST-API-KEY
{
    "message": (required, string) the status of the export, returns 'success' when completed without errors,
    "object_prefix": (required, string) the filename prefix that will be used for the JSON file produced by this export, e.g., 'bb8e2a91-c4aa-478b-b3f2-a4ee91731ad1-1464728599',
    "url" : (optional, string) the URL where the segment export data can be downloaded if you do not have your own S3 credentials
}
```

Once made available, the URL will only be valid for a few hours. As such, we highly recommend that you add your own S3 credentials to Braze.

## Sample user export file output

User export object (we will include the least data possible - if a field is missing from the object it should be assumed to be null, false, or empty):

```json
{
    "created_at" : (date),
    "external_id" : (string),
    "user_aliases" : [
      {
        "alias_name" : (string),
        "alias_label" : (string)
      }
    ],
    "braze_id": (string),
    "random_bucket" : (int)
    "first_name" : (string),
    "last_name" : (string),
    "email" : (string),
    "dob" : (string) date for the user's date of birth,
    "home_city" : (string),
    "country" : (string) ISO-3166-1 alpha-2 standard,
    "phone" : (string),
    "language" : (string) ISO-639 ISO-639-1 standard,
    "time_zone" : (string),
    "last_coordinates" : (array of float) [lon, lat],
    "gender" : (string) "M" | "F",
    "total_revenue" : (float),
    "attributed_campaign" : (string),
    "attributed_source" : (string),
    "attributed_adgroup" : (string),
    "attributed_ad" : (string),
    "push_subscribe" : (string) "opted_in" "push_opted_in_at" | "subscribed" | "unsubscribed" "push_unsubscribed_at",
    "email_subscribe" : (string) "opted_in" "email_opted_in_at" | "subscribed" | "unsubscribed" "email_unsubscribed_at",
    "custom_attributes" : (object) custom attribute key-value pairs,
    "custom_events" : [
        {
            "name" : (string),
            "first" : (string) date,
            "last" : (string) date,
            "count" : (int)
        },
        ...
    ],
    "purchases" : [
        {
            "name" : (string),
            "first" : (string) date,
            "last" : (string) date,
            "count" : (int)
        },
        ...
    ],
    "devices" : [
        {
            "model" : (string),
            "os" : (string),
            "carrier" : (string),
            "idfv" : (string) only included for iOS devices,
            "idfa" : (string) only included for iOS devices when IDFA collection is enabled,
            "google_ad_id" : (string) only included for Android devices when Google Play Advertising Identifier collection is enabled,
            "roku_ad_id" : (string) only included for Roku devices,
            "windows_ad_id" : (string) only included for Windows devices,
            "ad_tracking_enabled" : (bool)
        },
        ...
    ],
    "push_tokens" : [
        {
            "app" : (string) app name,
            "platform" : (string),
            "token" : (string),
            "device_id": (string),
            "notifications_enabled": (bool)
        },
        ...
    ],
    "apps" : [
        {
            "name" : (string),
            "platform" : (string),
            "version" : (string),
            "sessions" : (string),
            "first_used" : (string) date,
            "last_used" : (string) date
        },
        ...
    ],
    "campaigns_received" : [
        {
            "name" : (string),
            "last_received" : (string) date,
            "engaged" : {
                "opened_email" : (bool),
                "opened_push" : (bool),
                "clicked_email" : (bool),
                "clicked_in_app_message" : (bool)
            },
            "converted" : (bool),
            "api_campaign_id" : (string),
            "variation_name" : (optional, string) exists only if it is a multivariate campaign,
            "variation_api_id" : (optional, string) exists only if it is a multivariate campaign,
            "in_control" : (optional, bool) exists only if it is a multivariate campaign
        },
        ...
    ],
    "canvases_received": [
        {
            "name": (string),
            "api_canvas_id": (string),
            "last_received_message": (string) date,
            "last_entered": (string) date,
            "variation_name": (string),
            "in_control": (bool),
            "last_exited": (string) date,
            "steps_received": [
                {
                    "name": (string),
                    "api_canvas_step_id": (string),
                    "last_received": (string) date
                },
                {
                    "name": (string),
                    "api_canvas_step_id": (string),
                    "last_received": (string) date
                },
                {
                    "name": (string),
                    "api_canvas_step_id": (string),
                    "last_received": (string) date
                }
            ]
        },
        ...
    ],
    "cards_clicked" : [
        {
            "name" : (string)
        },
        ...
    ]
}
```

{% alert tip %}
For help with CSV and API exports, visit [Export troubleshooting]({{site.baseurl}}/user_guide/data_and_analytics/export_braze_data/export_troubleshooting/).
{% endalert %}

{% endapi %}
