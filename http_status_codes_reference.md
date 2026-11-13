# HTTP Status Codes Reference Guide

A quick reference guide for standard HTTP status codes, complete with one-line descriptions and practical examples.

---

## 1xx: Informational
* **100 Continue:** Server received request headers and the client can proceed to send the body.  
  *Example:* Uploading a large video file in chunks.
* **101 Switching Protocols:** Server agrees to switch protocols as requested by the client.  
  *Example:* Upgrading an HTTP connection to a WebSocket.

---

## 2xx: Success
* **200 OK:** Request succeeded and the server returned the requested data.  
  *Example:* Loading a standard webpage successfully.
* **201 Created:** Request succeeded and a new resource was successfully created.  
  *Example:* Successfully signing up for a new user account.
* **204 No Content:** Request succeeded but there is no payload body to return.  
  *Example:* Successfully deleting a database record via an API.

---

## 3xx: Redirection
* **301 Moved Permanently:** Resource has been permanently relocated to a new URL.  
  *Example:* Redirecting users from an old domain (`oldsite.com`) to a new one (`newsite.com`).
* **302 Found:** Resource temporarily resides under a different URL.  
  *Example:* Redirecting an unauthenticated user to a login page.
* **304 Not Modified:** Resource has not changed since the last request, using the cached version.  
  *Example:* Loading a cached CSS or image file to save bandwidth.

---

## 4xx: Client Error
* **400 Bad Request:** Server cannot process the request due to invalid syntax or malformed data.  
  *Example:* Sending malformed JSON in an API POST request.
* **401 Unauthorized:** Request lacks valid authentication credentials.  
  *Example:* Trying to access a private dashboard without logging in.
* **403 Forbidden:** Server understands the request but refuses to authorize it due to missing permissions.  
  *Example:* Trying to access an admin settings page as a regular user.
* **404 Not Found:** Server cannot find the requested resource.  
  *Example:* Visiting a broken or deleted URL link.
* **405 Method Not Allowed:** The HTTP method used is not supported by the endpoint.  
  *Example:* Using a `DELETE` request on a read-only endpoint.
* **409 Conflict:** Request could not be completed because of a conflict in resource state.  
  *Example:* Registering an account with an email address that already exists.
* **429 Too Many Requests:** User has sent too many requests in a given time frame.  
  *Example:* Triggering API rate-limits after making 100 rapid requests.

---

## 5xx: Server Error
* **500 Internal Server Error:** Generic error indicating the server encountered an unexpected condition.  
  *Example:* A fatal runtime crash in backend application code.
* **502 Bad Gateway:** Server acting as a gateway received an invalid response from an upstream server.  
  *Example:* Nginx proxy failing to reach the underlying Node.js app server.
* **503 Service Unavailable:** Server is temporarily overloaded or down for maintenance.  
  *Example:* An e-commerce site crashing under high flash-sale traffic.
* **504 Gateway Timeout:** Server acting as a gateway did not receive a timely response from an upstream server.  
  *Example:* A slow database query causing a proxy timeout.