---
title: Stats API
layout: post
category: For Developers
description: Want to export your statistics out of Wistia? We make it easy to grab the stats for an individual video at a time in a .csv file.
footer: 'for_developers'
---

Wistia provides a simple API to access individual viewer data for Wistia videos embedded on your website.  This data is provided in a comma-separated value (CSV) format.  In the CSV file, each line represents one view of the video.

The URL used to export analytics for a video is of the form:

<div class="code">https://&lt;account&gt;.wistia.com/stats/medias/&lt;id&gt;/events.&lt;format&gt;</div>

Here <span class="code">&lt;account&gt;</span> is your Wistia account key used in all URLs which reference your account.  <span class="code">&lt;id&gt;</span> is the Wistia id number of the video. <span class="code">&lt;format&gt;</span> can be either csv for json.

The easiest way to construct this base URL is, once you you are logged in to your Wistia account, go to Stats --&gt; Trends and then chose the video you wish to export stats for.  The URL displayed in the address bar will be of the form:

<div class="code">http://&lt;account&gt;.wistia.com/stats/medias/&lt;id&gt;</div>

Simply append <span class="code">/events.csv</span> and this will form your base URL.

All filter options are added via query string.  The query string parameters and possible values are:


*  **filter=view** This will show only events where the play button was pressed by the user.  If this option is not used, all events (included those where the video was not played) will be returned.
*  **limit** An integer that specifies the number of event records to return.  This has an upper limit of 1000.
*  **offset** An integer that specifies an offset in the total set to allow the caller to retrieve events in the middle of the total set.
*  **start_date** A date *(in the form of yyyy-mm-dd)* which specifies the earliest date for which to retrieve events.
*  **end_date** A date *(in the form of yyyy-mm-dd)* which specifies the latest date for which to retrieve events.

Events are always returned in descending order of the date on which they were created.  This means that the most recent events will be returned first.

Example:  To retrieve the 100 most recent play events for media #12345 in my Wistia account named **''foo''** starting on March 1, 2010, the request would be:

<div class="code"><pre>https://foo.wistia.com/stats/medias/12345/events.csv?filter=view&start_date=2010-03-01&limit=100</pre></div>

## Authentication

All analytics API requests must be authenticated.  These requests are authenticated using Wistia user credentials for a user that has access to the specified resource.

To access the data programmatically, Wistia uses HTTP basic authentication.  HTTP basic authentication requires a username and password.  For this API, the username is the email address of the Wistia manager that has access to the stats for this video and the password is the password that manager uses to log in to Wistia.

If you are using a tool or method that passes HTTP basic authentication credentials in the URL rather than inserting them programmatically into header of the request, you will need to URL encode the ** @ ** in the email address of the username.  To do this, replace the ** @ ** in the email address with **%40** .

As always, it is recommend that you use HTTPS for these requests as using plain HTTP will result in passing Wistia credentials in plain-text.


## Examples

To get the latest 100 play events for a media in JSON format:

	
<pre><code class="language-markup">curl -u <email>:<password> https://<account>.wistia.com/stats/medias/<media id>/events.json?filter=view&limit=100</code></pre>


To get the play events for the first day of March, 2010 in CSV format:

	
<pre><code class="language-markup">curl -u <email>:<password> https://<account>.wistia.com/stats/medias/<media id>/events.csv?filter=view&limit=1000&start_date=2010-03-01&end_date=2010-03-01 > 2010-03-01_events.csv</code></pre>
