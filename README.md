# REST API Spec
Simple specification/details for a clean and simple API setup. These "rules" are my own and in no way an official spec on how you should do it.

## Singular Object Names
Let's say we're creating an API for products. For the endpoint I like to use singular object names, eg "product" in stead of "products". So the endpoints would be.

* `GET /api/product` Get all entities.
* `GET /api/product/{id}` Get a specific entity.
* `POST /api/product` Create a new entity.
* `POST /api/product/{id}` Update an existing entity.
* `DELETE /api/product/{id}` Delete an existing entity.

Note the use of "product" even on the get-all endpoint.

## HTTP Status Codes
Here is a list of all the status codes I use in my projects. Instead of returning the status code in the body, I like to use the already existing HTTP status codes.

* `200 Ok` Everything is ok and some data will be returned in the body.
* `201 Created` An entity was created and will be returned in the body.
* `204 No Content` Everything is ok, but no data will be returned. This works well for updating or deleting existing entities.
* `400 Bad Request` The client sent an invalid request, such as lacking required body or parameter. The return body will include details about the error that was made.
* `401 Unauthorized` The client failed to authenticate properly.
* `403 Forbidden` The client has authenticated properly but does not have access to the requested resource.
* `404 Not Found` The requested resource was not found.
* `429 Too Many Requests` The client has sent too many requests in a given amount of time.
* `500 Internal Server Error` Something erroneous happened on the server.

## Return Data
In the example above, using product, the following requests would return `204 No Content` if everything went ok:
* `POST /api/product/{id}` Update an existing entity.
* `DELETE /api/product/{id}` Delete an existing entity.

The following request will return `201 Created` with a body:
* `POST /api/product` Create a new entity.

The following requests will return `200 Ok` with a body:
* `GET /api/product` Get all entities.
* `GET /api/product/{id}` Get a specific entity.

Getting a single product will return the complete entity, while getting all products will return a mixed content like so:
```json
{
	"data": [
		{
			"id": 1
		},
		{
			"id": 2
		}
	],
	"items": {
		"perPage": 1000,
		"returned": 2,
		"total": 5005
	},
	"links": {
		"firstPage": "https://api.dev/api/product",
		"lastPage": "https://api.dev/api/product?skip=5000",
		"nextPage": "https://api.dev/api/product?skip=1000",
		"prevPage": null
	}
}
```

## Errors
How we return an error is important. I like to use the HTTP status code to indicate what happened with the request.

401, 403, and 500 can be tricky when it comes to details. Normally you don't wanna reveal any information about your users, so I just return the status code and nothing else. 500 *sometimes* needs to return more info which can be relevant to the client, but that's very rare.

400, and sometimes 404, needs more details. I've become a fan of the following structure for an Bad Request error payload:
```json
{
	"code": "/errors/missing-argument",
	"message": "Missing argument 'FirstName'.",
	"detail": "The argument 'FirstName' is missing from the request body.",
	"traceId": "cefaa0cf-0539-4db6-8b65-0a857079882c"
}
```

That way you get a code which can easily be read by some automation, a brief message with the error, a detailed message with more information, and a trace id that can be used in debugging.