---
title: API versioning
date: 2018-08-24 16:00:00 +0200
categories: rest api
---

It is pretty standard to version APIs by including the version in the URLs like `/api/v1/posts`.
In my real-life, experience working at multiple companies on various production APIs I never got to see `v2` or later versions of the API.
What happened instead was that existing endpoints were extended in a backwards compatible way and new endpoints were created. And in rare cases endpoints were deprecated. And the reason is simple: API versioning is expensive. Also, it usually is not needed.

## Safe changes

### More data

Accepting more data and responding with more data is backwards compatible. So if you used to return

```json
{
  "name": "My Post",
  "date": "2018-08-24 16:00:00 +0200"
}
```

and later find out you also want to return author data

```json
{
  "name": "My Post",
  "date": "2018-08-24 16:00:00 +0200",
  "author_name": "Orhan Toy"
}
```

then this is totally fine and should not break clients.

### New type of interaction

Exposing new functionality with new endpoints via your API will also not break clients.

### Changing data type

The data type could change if you used to return

```json
{
  "name": "My Post",
  "date": "2018-08-24 16:00:00 +0200",
  "category": "rest-api"
}
```

and now you want to support multiple categories like

```json
{
  "name": "My Post",
  "date": "2018-08-24 16:00:00 +0200",
  "categories": ["rest-api", "json"]
}
```

You could solve this by returning both `category` and `categories` in the response. The `category` could just be set to any of the associated categories and the response would still be backwards compatible.

## Unsafe changes

### Remove interaction

Well, if you want to remove some API endpoint, it would not make a huge difference whether you remove the endpoint from the current API version or create a new API version without the endpoint.

Logging which clients/tokens use the deprecated endpoint and reaching out to the company would be the best way to handle this scenario.

### Changing validation rules

If you at some point decide that the `date` parameter should now be required, this would not be backwards compatible.

I think something like this could be handled by introducing a different code path for the same endpoint that tests a HTTP header, e.g. `X-Require-Post-Date: true`. Ideally you would log which clients/tokens have started using the header and which clients who have not migrated yet so you can reach out to the affected companies. Ultimately you would most likely enforce the new implementation and just ignore the header.

In this case something like API-versioning makes a lot of sense but if you think it through it still comes at a cost:

- When reading data you can't tell if the `date` will be there or not, it depends on the API version.
- The domain model and database constraints can not actually be changed because you would always have to support the nullable `date`.

## Conclusion

The overall point I'm trying to make is that it should **not** be the default from the beginning when building an API to assume versioning will be needed. It would be like namespacing all your domain logic (classes and modules) with a version number *in case* of you one day need to rethink some concepts. Concepts do change, new concepts get introduced and deprecated concepts get removed but versioning your API is like saying that nothing will change for version X. But the cost of keeping this promise will usually be too high and not worth it, especially for smaller teams.

*This post talks about REST APIs. GraphQL has an interesting take on API versioning which in some ways are similar to what I'm describing.*
