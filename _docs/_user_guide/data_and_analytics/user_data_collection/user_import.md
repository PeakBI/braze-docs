---
nav_title: User Import
article_title: User Import
page_order: 4
page_type: reference
description: "This reference article covers the topic of how to import users into your Braze dashboard and best practices."

---
# User import

There are two approaches for importing customer data into your Braze dashboard: REST API and CSV.

{% alert important %}
Braze does not sanitize HTML data during ingestion time. This means that script tags must be stripped for all import data meant for web personalization. Skip to the [HTML data stripping](#html-data-stripping) section to read more.
{% endalert %}

## REST API

You can use Braze’s User Track REST API endpoint to record custom events, user attributes, and purchases for users. See [User Track Endpoint][12] for more information.

## CSV

You can also upload and update user profiles via CSV files from the **User Import** page. This feature supports recording and updating user attributes such as first name and email, in addition to custom attributes such as shoe size. There are two different ways you can approach a CSV import, depending on if your users have an `external_id` or not.

### Importing with external ID

When importing your customer data, you'll need to specify each customer’s unique identifier, also known as `external_id`. Before starting your CSV import it’s important to understand from your engineering team how users will be identified in Braze. Typically this would be a database ID used internally. This should align with how users will be identified by the Braze SDK on mobile and web, and ensures that each customer will have a single user profile within Braze across their devices. Read more about Braze’s [user profile lifecycle][13].

When you provide an `external_id` in your import, Braze will update any existing user with the same `external_id` or create a newly identified user with that `external_id` set if one is not found.

<i class="fas fa-file-download"></i> Download: [CSV Import Template][template]

### Importing with user alias

To target users who don't have an `external_id`, you can import a list of users with user aliases. An alias serves as an alternative unique user identifier, and can be helpful if you are trying to market to anonymous users who haven't signed up or made an account with your app.

If you are uploading or updating user profiles that are alias only, you must have the following two columns in your CSV:

- `user_alias_name`: A unique user identifier; an alternative to the `external_id`.
- `user_alias_label`: A common label by which to group user aliases.

![User Alias Import CSV Example][8]{: style="max-width:80%" }

When you provide both a `user_alias_name` and `user_alias_label` in your import, Braze will update any existing user with the same `user_alias_name` or create a newly identified user with that `user_alias_name` set if one is not found.

{% alert important %}
You can't use a CSV import to update an existing user with a `user_alias_name` if they already have an `external_id`. Instead, this will create a new user profile with the associated `user_alias_name`. To associate an alias-only user with an `external_id`, use the [Identify Users]({{site.baseurl}}/api/endpoints/user_data/post_user_identify/) endpoint.
{% endalert %}

<i class="fas fa-file-download"></i> Download: [CSV Alias Import Template][template_alias]

### Importing with Braze ID

To update existing user profiles in Braze using an internal Braze ID value instead of an `external_id` or `user_alias_name` / `user_alias_label` value, you can do this by specifying `braze_id` as a column header.

This can be helpful if you have exported user data from Braze via our CSV export option within segmentation and wish to add a new custom attribute to these existing users.

{% alert important %}
You can't use a CSV import to create a new user using `braze_id`. This method can only be used to update pre-existing users within the Braze platform.
{% endalert %}

{% alert tip %}
The `braze_id` value may be labelled as `Appboy ID` in CSV exports from the Braze dashboard. This ID will be the same as the `braze_id` for a user and therefore you can simply rename this column to `braze_id` when you re-import the CSV.
{% endalert %}

### Constructing your CSV

There are several data types in Braze. When importing or updating user profiles via CSV, you can create or update default user attributes or custom attributes.

- Default user attributes are reserved keys in Braze. For example, `first_name` or `email`.
- Custom attributes are custom to your business. For example, a travel booking app may have a custom attribute called `last_destination_searched`.

{% alert important %}
When importing customer data, the column headers you use must exactly match the spelling and capitalization of default user attributes. Otherwise, Braze will automatically create a custom attribute on that user’s profile.
{% endalert %}

Braze accepts user data in the standard CSV format from files up to 100MB in size. Refer to the preceding sections on importing for downloadable CSV templates.

### Data point considerations

Each piece of customer data imported via CSV will overwrite the existing value on user profiles and will count as a data point, except for external IDs and blank values. 

- External IDs uploaded via CSV will not consume data points. If you are uploading a CSV to segment existing Braze users by uploading only external IDs, this can be done without consuming data points. If you were to add additional data like user email or phone number in your import, that would overwrite existing user data, consuming your data points.
- Blank values will not overwrite existing values on the user profile, and you do not need to include all existing user attributes in your CSV file.

{% alert important %}
Setting `language` or `country` on a user via CSV import or API will prevent Braze from automatically capturing this information via the SDK.
{% endalert %}

### Default user data column headers

| USER PROFILE FIELD | DATA TYPE | INFORMATION | REQUIRED |
|---|---|---|---|
| `external_id` | String | A unique user identifier for your customer. | Yes, see the following note |
| `user_alias` | String | A unique user identifier for anonymous users. An alternative to the `external_id`. | No, see the following note |
| `user_alias_label` | String | A common label by which to group user aliases. | Yes if `user_alias` is used |
| `first_name` | String | The first name of your users as they have indicated (e.g., `Jane`). | No |
| `last_name` | String | The last name of your users as they have indicated (e.g., `Doe`). | No |
| `email` | String | The email of your users as they have indicated (e.g., `jane.doe@braze.com`). | No |
| `country` | String | Country codes must be passed to Braze in the ISO-3166-1 alpha-2 standard (e.g., `GB`). | No |
| `dob` | String | Must be passed in the format “YYYY-MM-DD” (e.g., `1980-12-21`). This will import your user’s Date of Birth and enable you to target users whose birthday is “today”. | No |
| `gender` | String | “M”, “F”, “O” (other), “N” (not applicable), “P” (prefer not to say), or nil (unknown). | No |
| `home_city` | String | The home city of your users as they have indicated (e.g., `London`). | No |
| `language` | String | Language must be passed to Braze in the ISO-639-1 standard (e.g., `en`). <br>Refer to our [list of accepted languages][1]. | No |
| `phone` | String | A telephone number as indicated by your users, in `E.164` format (e.g., `+442071838750`). <br> Refer to [User Phone Numbers][2] for formatting guidance. | No |
| `email_open_tracking_disabled` | Boolean | true or false accepted.  Set to true to disable the open tracking pixel from being added to all future emails sent to this user.   | No |
| `email_click_tracking_disabled` | Boolean | true or false accepted.  Set to true to disable the click tracking for all links within a future email, sent to this user. | No |
| `email_subscribe` | String | Available values are `opted_in` (explicitly registered to receive email messages), `unsubscribed` (explicitly opted out of email messages), and `subscribed` (neither opted in nor out). | No |
| `push_subscribe` | String | Available values are `opted_in` (explicitly registered to receive push messages), `unsubscribed` (explicitly opted out of push messages), and `subscribed` (neither opted in nor out). | No |
| `time_zone` | String | Time zone must be passed to Braze in the same format as the IANA Time Zone Database (e.g., `America/New_York` or `Eastern Time (US & Canada)`).  | No |
| `date_of_first_session` <br><br> `date_of_last_session`| String | May be passed in one of the following ISO8601 formats: <br> - "YYYY-MM-DD" <br> - "YYYY-MM-DDTHH:MM:SS+00:00" <br> - "YYYY-MM-DDTHH:MM:SSZ" <br> - "YYYY-MM-DDTHH:MM:SS" (e.g., `2019-11-20T18:38:57`) | No |
{: .reset-td-br-1 .reset-td-br-2 .reset-td-br-3  .reset-td-br-4}


{% alert note %}
While `external_id` itself is not mandatory, you **must** include one of these fields:
- `external_id` - A unique user identifier for your customer <br> - OR -
- `braze_id` - A unique user identifier pulled for existing Braze users <br> - OR -
- `user_alias` - A unique user identifier for an anonymous user
{% endalert %}


### Importing custom data via CSV

Any headers that do not exactly match default user data will create a custom attribute within Braze.

The following data types are accepted in User Import:
- Datetime (Must be stored in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format)
- Boolean (TRUE/FALSE)
- Number (Integer or Float with no spaces or commas, floats must use a period ‘.’ as the decimal separator)
- String (no commas)
- Blank (Blank values will not overwrite existing values on the user profile, and you do not need to include all existing user attributes in your CSV file.)

{% alert important %}
Arrays, push tokens, and custom event data types are not supported in User Import.
Especially for arrays, commas in your CSV file will be interpreted as a column separator, so any commas in values will cause errors parsing the file.

For uploading these kinds of values, use the [User Track Endpoint]({{site.baseurl}}/developer_guide/rest_api/user_data/#user-track-endpoint).
{% endalert %}

### Importing a CSV

To import your CSV file, navigate to the **User Import** page under the Users section. In the lower text box, **Recent Imports**, there will be a table that lists up to twenty of your most recent imports, their file names, number of lines in the file, number of lines successfully imported, total lines in each file, and the status of each import.

The upper box, **Import CSV**, will contain importing directions and a button to begin your import. Click **Select CSV File** and select your file of interest, then click **Start Upload**. Braze will upload your file and check the column headers as well as the data types of each column. 

To download a CSV template, refer to the sections [Import with External ID](#import-with-external-id) or [Import with User Alias](#import-with-user-alias) on this page.

{% alert important %}
CSV imports are case sensitive. This means capital letters in CSV imports will write the field as a custom attribute instead of a standard one. For example, "email" is correct, but "Email" would be written as a custom attribute.
{% endalert %}

![CSV Import][3]

Once the upload is complete, you will see a modal window with a table previewing the contents of your file. All the information in this table is based on the values in the top few rows of your CSV file. For column headers, default attributes will be written in normal text, while custom attributes will be italicized and have their type noted in parentheses. There will also be a short summary of your file at the top of the pop-up.

You can import more than one CSV at the same time. CSV imports will run concurrently, and as such the order of updates is not guaranteed to be serial. If you require CSV imports to run one after another, you should wait until a CSV import has finished before uploading a second one.

If Braze notices something malformed in your file during the upload, errors will be shown with the summary. A file can be imported with errors, but an import can't be canceled or rolled-back once started. Review the preview, and if you find any errors, cancel the import and modify your file. It's important to examine the full CSV file before upload, as Braze doesn't scan every row of the input file for the preview. This means errors can exist which Braze doesn't catch while generating this preview.

Malformed rows and rows lacking an external ID will not be imported. All other errors can be imported, but may interfere with filtering when creating a segment. For more information, skip to the [Troubleshooting](#troubleshooting) section.

![CSV upload completed with errors involving mixed data types in a single column][4]{: style="max-width:70%"}

{% alert warning %}
Errors are based solely on data type and file structure. For example, a poorly formatted email address would still be imported as it can still be parsed as a string.
{% endalert %}

When you're satisfied with the upload, start the import. The pop-up will close and the import will begin in the background. You can track its progress on the **User Import** page, which will refresh every 5 seconds, or at the press of the refresh button in the **Recent Imports** box.

Under **Lines Processed**, you will see the progress of the import; the status will change to Complete when finished. You can still use the rest of the Braze dashboard during the import, and you'll receive notifications when the import begins and ends.

If the import process runs into an error, a yellow warning icon will be displayed next to the total number of lines in the file. You can hover over the icon to see details into why certain lines failed. Once the import is complete, all data will be added to existing profiles, or new profiles will be created.

## Segmenting

User import creates and updates user profiles, and can also be used to create segments. To create a segment, select **Automatically generate a segment from the users who are imported from this CSV** before starting the import.

You can set the name of the segment or accept the default, which is the name of your file. Files that were used to create a segment will have a link to view the segment once the import has been completed.

The filter used to create the segment selects users who were created or updated in a selected import and is available with all other filters in the edit segment page.

## HTML data stripping

Braze does not sanitize HTML data during ingestion time. When importing data into Braze, specifically meant for personalization usage in a web browser, ensure that it is stripped of HTML, JavaScript, or any other script tag that potentially could be leveraged maliciously when rendered in a web browser.  

Alternatively, for HTML, you can use Braze's Liquid filters (`strip_html`) to HTML escape rendered text. For example:

{% tabs local %}
{% tab Input %}
{% raw %}
```liquid
{{ "Have <em>you</em> read <strong>Ulysses</strong>?" &#124; strip_html }}
```
{% endraw %}
{% endtab %}
{% tab Output %}
{% raw %}
```liquid
Have you read Ulysses?
```
{% endraw %}
{% endtab %}
{% endtabs %}

## Troubleshooting {#troubleshooting}

### Missing rows

There are a few reasons why the number of users imported might not match the total rows in your CSV file:

- **Duplicate external IDs:** If there are duplicate external ID columns, then this may cause malformed or unimported rows even if the rows are correctly formatted. In some cases this may not report a specific error. Check if there are any duplicate external IDs in your CSV. If so, remove the duplicates and try uploading again.
- **Accented characters:** Your CSV may have names or attributes that include accents. Ensure your file is UTF-8 encoded to prevent any issues.

### Malformed row

There must be a header row in order to properly import data. Each row must have the same number of cells as the header row. Rows whose length has more or fewer values than the header row will be excluded from the import. Commas in a value will be interpreted as a separator and can lead to this error being thrown. Additionally, all data must be UTF-8 encoded.

If your CSV file has blank rows and imports less rows than the total lines in the CSV file, this may not indicate a problem with the import since the blank rows wouldn't need to be imported. Check the number of lines that were correctly imported and make sure it matches the number of users you're attempting to import.

### Multiple data types

Braze expects each value in a column to be of the same data type. Values that do not match their attribute’s data type will cause errors in segmenting.

### Incorrectly formatted dates

Dates not in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format will not be read as datetimes on import.

### String quotation

Values encapsulated in single (‘’) or double (“”) quotation marks will be read as strings on import.

### Data imported as custom attribute

If you are seeing a piece of default user data (e.g., `email` or `first_name`) imported as a custom attribute, check the case and spacing of your CSV file. For example, `First_name` would be imported as a custom attribute, while `first_name` would be correctly imported into the “first name” field on a user’s profile.

{% alert important %}
Braze will ban or block users with over 5 million sessions ("dummy users") and no longer ingest their SDK events. For more information, refer to [Spam blocking]({{site.baseurl}}/user_guide/data_and_analytics/user_data_collection/user_archival/#spam-blocking).
{% endalert %}

[1]: {{site.baseurl}}/user_guide/data_and_analytics/user_data_collection/language_codes/
[2]: {{site.baseurl}}/user_guide/message_building_by_channel/sms/phone_numbers/user_phone_numbers/
[3]: {% image_buster /assets/img/importcsv.png %}
[4]: {% image_buster /assets/img/importcsv2.png %}
[7]: {% image_buster /assets/img/segment-imported-users.png %}
[8]: {% image_buster /assets/img_archive/user_alias_import_1.png %}
[12]: {{site.baseurl}}/developer_guide/rest_api/user_data/#user-track-endpoint
[13]: {{site.baseurl}}/user_guide/data_and_analytics/user_data_collection/user_profile_lifecycle/
[errors]:#common-errors
[template]: {% image_buster /assets/download_file/braze-user-import-template-csv.xlsx %}
[template_alias]: {% image_buster /assets/download_file/braze-user-import-alias-template-csv.xlsx %}
