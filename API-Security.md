
### Encryption

All API requests must use HTTPS to prevent man-in-the-middle attacks.

### Client Identity

Each client who is given permission to use the API will be given a CLIENT_ID
and a SECRET_KEY.

### Requests

Requests will be made using GET, POST, PUT/PATCH or DELETE.

For POST and PUT/PATCH requests, the body must be in the JSON format.

The CLIENT_ID must be included in all request.

### Tamper Proofing

All API requests must include (append if a GET request, put in the body if a
POST request) a SHA-2 (SHA-512) one way hash.

The one way hash is constructed of the data fields that are sent with the
requests, as well as the SECRET_KEY.

The CLIENT_ID must **not** be included in the hash.

When the request arrives at the server the following will occur:  
1.  The SECRET_KEY will be read from the database using the provided client's
    CLIENT_ID.
1.  A new hash will be constructed using the request fields and the SECRET_KEY
    read from the database.
1.  The sent hash and the generated hash will be compared. If they are
    identical then the request will be processed.

### API Versioning

The version must be included in the HTTP Accept header of the request.

### Response Headers

The 2xx response status codes indicate a successful request and the 4xx
response status codes indicate a failed request.

For example a successful POST request would typically return the headers:  
```
Status: 201 Created
Location: <location of the resources>
```

And return a JSON body of:  
```
{
  "created_at": "2013-09-20 12:06:57",
  "object_id": "1234"
}
```

Beside the 4xx response status code, a failed request would typically return a
JSON body:  
```
{
  "code": 100,
  "error": "invalid"
}
```

