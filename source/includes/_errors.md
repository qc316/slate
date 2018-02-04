# Errors

This section describes all the errors that you can encounter with our API and their meaning.

Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid. Check the sent parameters
401 | Unauthorized -- Your JWT is either invalid or expired.
404 | Not Found -- The specified endpoint does not exist.
406 | Not Acceptable -- You requested a format that isn't json.
500 | Internal Server Error -- We had a problem with our server. Please try again later or contact us.
503 | Service Unavailable, please try again later or contact us.

<aside class="notice">
If you received a 401 error code, you just have to log in again to get a valid JWT
</aside>
