---
layout: post
title: The Wistia Upload API
api: true
special_category_link: developers
description: A simple mechanism for getting your videos into Wistia.
category: Developers
footer: 'for_developers'
post_intro: "<p>The Upload API is the best way to programmatically get new videos and files into your Wistia account.</p><p>If you are looking to have site visitors upload content, you should <a href='/doc/account-setup#api_access'>create a new access token</a> with upload permissions for that.</p>"
---

## Uploading to Wistia

To upload a file from your computer, supply the required parameters and
**POST** your media file to **https://upload.wistia.com/** as multipart-form
encoded data.

Uploaded media will be visible immediately in your account, but may require
processing (as is the case for uploads in general).

## Importing Web/FTP Content to Wistia

To import a file from a web or FTP server, supply the required parameters as a
standard form-url encoded **POST** to **https://upload.wistia.com/**.

Imported media will always require some processing time, which varies depending
on the file size and connection speed.

## There's a Gem For That

To make using the Upload API even easier, we created a [Ruby
gem](https://github.com/wistia/wistia-uploader) that lets you use it via the
command line! Even if you're not a Ruby developer, you can use this command line
tool for uploading to Wistia.

## Authentication

* All upload requests must use **SSL** (*https://*, not *http://*).
* The required *api_password* parameter will be used for authentication.

## The Request

<code class="full_width">POST https://upload.wistia.com</code>

All parameters (with the exception of *file*) may be encoded into the request
body or included as part of the query string.

The *file* parameter must be multipart-form encoded into the request body.

Parameter   | Description
------------|-------------
api_password  | **Required unless access_token is specified**. A 40 character hex string. This parameter can be found on your [API access page]({{ '/account-setup#api_password_and_public_token' | post_url }}).
access_token  | **Required unless api_password is specified**. The token you received from authenticating via [OAuth2]({{ '/oauth2' | post_url }}).
file          | **Required unless `url` is specified**. The media file, multipart-form encoded into the request body.
url           | **Required unless `file` is specified**. The web or FTP location of the media file to import.
project_id    | The hashed id of the project to upload media into. If omitted, a new project will be created and uploaded to. The naming convention used for such projects is *Uploads_YYYY-MM-DD*.
name          | A display name to use for the media in Wistia. If omitted, the filename will be used instead.
description   | A description to use for the media in Wistia. You can use basic HTML here, but note that both HTML and CSS will be sanitized.
contact_id    | A Wistia contact id, an integer value. If omitted, it will default to the contact_id of the account's owner.



## Examples

**Uploading a media file with cURL**

<code class="full_width">$ curl -i -F api_password=&lt;YOUR_API_PASSWORD&gt; -F file=@&lt;LOCAL_FILE_PATH&gt; https://upload.wistia.com/</code>

We expect a multipart/form-data POST to the server, with both **file** and
**api_password** params.

**Importing a media file via URL with cURL**

<code class="full_width">$ curl -i -d "api_password=&lt;YOUR_API_PASSWORD&gt;&amp;url=&lt;REMOTE_FILE_PATH&gt;" https://upload.wistia.com/</code>

We expect an `application/x-www-form-urlencoded` POST to the server, with both
**url** and **api_password** params.

### Using the OAuth2 Access Token

If you are using the **access_token** param with [OAuth2]({{ '/oauth2' | post_url }}),
it should be appended to the URL as a query parameter, like:

    https://upload.wistia.com/?access_token=myaccesstoken

If you are importing the media file via URL, you can also include the
**access_token** as a POST param.

## Response Format

For successful uploads, the Upload API will respond with an HTTP-200 and the
body will contain a JSON object.

  * **HTTP 200** *[application/json]* Success. A JSON object with media details.
  * **HTTP 400** *[application/json]* Error. A JSON object detailing errors.
  * **HTTP 401** *[text/html]* Authorization error. Check your api_password.

The most common error that all implementations should beware of is a 400 due 
to reaching the video limit of your account. Not all accounts have video
limits, but for those that do, you will receive a 400 response with JSON like:

    {
      "error": "This account has exceeded its video limit. Please upgrade to upload more videos."
    }


### Example Response

{% codeblock upload_response.json %}
{
  "id"=>2208087, 
  "name"=>"dramatic_squirrel.mp4", 
  "type"=>"Video", 
  "created"=>"2012-10-26T16:47:09+00:00", 
  "updated"=>"2012-10-26T16:47:10+00:00", 
  "duration"=>5.333000183105469, 
  "hashed_id"=>"gn69c10tqw",
  "progress"=>0.0,
  "thumbnail"=>
  {
    "url"=>"http://embed.wistia.com/deliveries/ffbada01610466e66f67a5dbbf473ed6574a6405.jpg?image_crop_resized=100x60", 
    "width"=>100, 
    "height"=>60
  }
}
{% endcodeblock %}

This data structure may change in future releases.

## Examples Using Ruby

All media uploaded via https://upload.wistia.com must be transferred as
multipart-form encoded data inside the body of an HTTP-POST. This can be
achieved in Ruby quite simply using the 'multipart-post' gem.

Installation:

<code class="full_width">$ gem install multipart-post</code>

### Example Code

{% codeblock upload_ruby_gem.rb %}
require 'net/http'
require 'net/http/post/multipart'

def post_video_to_wistia(name, path_to_video)
  uri = URI('https://upload.wistia.com/')

  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = true

  # Construct the request.
  request = Net::HTTP::Post::Multipart.new uri.request_uri, {
    'api_password' => '<API_PASSWORD>',
    'contact_id'   => '<CONTACT_ID>', # Optional.
    'project_id'   => '<PROJECT_ID>', # Optional.
    'name'         => '<MEDIA_NAME>', # Optional.

    'file' => UploadIO.new(
                File.open(path_to_video),
                'application/octet-stream',
                File.basename(path_to_video)
              )
  }

  # Make it so!
  response = http.request(request)

  return response
end
{% endcodeblock %}

## Upload Via FTP

Do you have an FTP client? One that is awesome? I use [Transmit](http://panic.com/transmit/) for Mac, or [Filezilla](https://filezilla-project.org/download.php) for PC. 

We've been quietly testing upload via FTP, for folks who want to move a large 
library over to Wistia. The bugs have not been worked out, and some of our requirements
means not all FTP clients can support it, but for certain use cases it may help.

Here's the workflow for Transit:

* Load up transmit, and enter the following settings:
  * Server: ftp.wistia.com
  * User Name: wistia
  * Password: your account's API password
  * and make sure to select *FTP with TLS/SSL* from the security options.
* Your projects list will appear. 
* Upload into the Project's folder, not the `_assets` dir  
<br>
<br>
Here's the workflow for Filezilla:

* Load up Filezilla, and use these settings:
  * Host: ftp.wistia.com
  * Protocol: FTP — File Transfer Protocol
  * Encryption: Require explicit FTP over TLS
  * Logon Type: Normal
  * User: wistia
  * Password: your account's API password
  * Your projects list will appear. 
  * Upload into the Project's folder, not the `_assets` dir

