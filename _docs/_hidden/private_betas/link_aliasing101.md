---
nav_title: Link Aliasing 101
permalink: /link_aliasing_101/
description: "This article describes what to expect when Link Aliasing is enabled, such as impacts on your workflow and what your links will look like."
hidden: true
---

# Link Aliasing 101

## What is Link Aliasing?

Link Aliasing is a new feature that allows marketers to create recognizable, user-generated names to identify links sent in email messages from Braze. These user-generated names are then made available for segmentation retargeting and action-based triggering, and in the reporting display of click information.

## What to expect

{%raw%}
When Link Aliasing is enabled, expect the following behaviors to occur:
1. All recognized links in an email are decorated with an additional query parameter `lid={{placeholder}}`, where `{{placeholder}}` is a unique Liquid-generated alphanumeric value. This occurs when the client either clicks the **Link Management** tab in the composer or clicks **Done** to navigate back to the message workflow.
2. New Content Blocks will have their links modified by appending `lid={{placeholder}}` to each link, where applicable. The `{{placeholder}}` value is resolved when inserted into an email message variant only. If the HTML content block is utilized in other channels, such as in-app messages, the `lid` query parameter will still be appended.
	- Content Blocks that were created before Link Aliasing enablement will not have their links recognized by the feature. Customers are encouraged to duplicate and create new Content Blocks that include the Link Aliasing query parameters.
{%endraw%}

{% alert note %}
Clients can also update their existing Content Block. However, only a maximum of 50 messages referencing that Content Block will be updated for Link Aliasing purposes.
{% endalert %}

### Messages with Link Templates
Messages which already had Link Templates enabled prior to Link Aliasing being enabled, will still continue to have their Link Templates applied.  For all messages that existed prior to Link Aliasing being enabled, the Link Templates will still honored, however on any edit, the customer is asked to re-apply Link Templates again from the Link Management tab.  This is because the Link Template edit screen is no longer be available since it is now provided under the Link Management tab.

## What use cases does this potentially unlock?

Clients can now distinguish between the same URL present in their email message by providing unique link alias names to each one (for example, "main1" and "main2", which both direct to https://www.braze.com). Clients can now retarget individual aliases that are clicked in segmentation.

{% alert note %}
There is a maximum of 100 aliases allowed for retargeting purposes at this time.
{% endalert %}

Clients can create action-based triggers based on an alias that is clicked as there's no limit here since action-based is triggered "in the moment" we receive a click event.

The Click table in the Engagement table will now be indexed by aliases rather than the top-level domain or domain and query parameters. This view allows for a clearer understanding of links based on the position in the email rather than URL level performance.

## What impact will it have on my workflow?
A recommended operation change is to ensure that clients review the **Link Management** tab as they are creating their email. This will ensure clients can quickly evaluate the links present within the email, add link templates, and provide a naming convention that works for segmentation and reporting purposes.

## Can you show me what my links will look like?
Braze looks for the anchor tag in the HTML to identify the link (`<a href=””> Link Text </a>`). Braze will then append based on the presence of a question  mark(?), which indicates that query parameters can be appended.  

Here are a few examples where Braze can safely append the `lid` query parameter:

{%raw%}
| Link in Email Body | Link with Aliasing| Logic |
|---|---|---|
| https://www.braze.com | https://www.braze.com?lid=slfdldtqdhdk | Braze inserts a question mark (?) and adds the first query parameter into the URL. |
| https://www.braze.com?utm_campaign=retention&utm_source=email | https://www.braze.com?utm_campaign=retention&utm_source=email&lid=0goty30mviyz | Braze detects other query parameters and appends `lid=` to the end of the URL. |
| `<a href="{{custom_attribute.{product_url}}?">` | `<a href=”{{custom_attribute.{product_url}}?lid=ac7a548g5kl7”>` | Braze recognizes that this is a URL and already has a question mark (?) present. Then, it appends the `lid` query parameter after the question mark. |
| https://www.braze.com#bookmark1?utm_source=email | https://www.braze.com?lid=eqslgd5a9m3y#bookmark1?utm_source=email | Braze expects the URL to use a standard structure where anchors (#) are present after a question mark (?).  Because Braze reads from left to right, we will append the question mark and `lid` value before the anchor. |
{: .reset-td-br-1 .reset-td-br-2 .reset-td-br-3}
{%endraw%}

## Can I enable this only for a particular message or app group?
Currently, this feature is enabled at a company level. Please ensure your organization is aware of this change before Braze enables the feature.

## How do I track or untrack a link?
Clients can track a link by using the checkbox located in the first column of the **Link Management** tab. Clients can track an unlimited number of links.

{% alert note %}
Braze only tracks up to the last 100 clicked link aliases at the profile level.
{% endalert %}

Each alias is considered unique to Braze, regardless of the name provided. This means even the same URL that is present in multiple variants will be considered on an individual basis.

To untrack a link, simply uncheck the checkbox or [archive the campaign]({{site.baseurl}}/user_guide/engagement_tools/campaigns/managing_campaigns/archiving_campaigns/#archiving-campaigns).

## I'm interested! How do I enable Link Aliasing?
This feature is still considered in beta, so Braze requires a beta agreement before enabling this feature. Please submit this [Link Aliasing acknowledgement form][1], and Braze will enable the feature during the next release window, which occurs weekly.

[1]: https://docs.google.com/forms/d/e/1FAIpQLSfoEXZ9hQfw61AykoKgp2wGtcWyFsjVGGltfTsF0HkNhdU1og/viewform
