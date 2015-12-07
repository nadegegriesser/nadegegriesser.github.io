---
layout: post
title: REST Client Browser Add-on
---

You can find REST Client Add-ons for most of the browsers. They propose a user friendly interface to generate, send REST requests and analyze the response. These are useful for punctual manual testing.

For example:

* Firefox : "RESTClient, a debugger for RESTful web services"
* Chrome : "Advanced REST client"

Let us use RESTClient to send requests to the basic REST service.

### List all the resources

We would like a result in JSON format, so we set the Accept header accordingly.

![GET /persons]({{ site.baseurl }}/images/firefox-rest-client/get-persons.png "GET /persons")

The Response Headers tab shows a Status Code 200 OK, the Response Body tabs (not displayed here) show an empty list.

### Create a resource

We send data in JSON format, so we set the Content-Type header accordingly.

We would like a result in JSON format, so we set the Accept header accordingly.

![POST /person]({{ site.baseurl }}/images/firefox-rest-client/post-person.png "POST /person")

The create operation has generated a unique identifier for the resource.

### Return a resource

We use the identifier previously generated to retrieve the resource.

We would like a result in JSON format, so we set the Accept header accordingly.

![GET /person]({{ site.baseurl }}/images/firefox-rest-client/get-person-after-create.png "GET /person")

### Update a resource

![PUT /person]({{ site.baseurl }}/images/firefox-rest-client/put-person.PNG "PUT /person")

### List all the resources

![GET /persons]({{ site.baseurl }}/images/firefox-rest-client/get-persons-after-create-and-update.PNG "GET /persons")

We get a list of resources with one element (the resource we updated previously).

### Delete a resource

No data sent, no response body expected so no need to set headers.

![DELETE /person]({{ site.baseurl }}/images/firefox-rest-client/delete-person.PNG "DELETE /person")

The Response Headers tab shows a Status Code 204 No Content
