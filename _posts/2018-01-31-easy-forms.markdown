---
title: "Easy forms"
date: 2018-01-31 20:00:00 +0100
categories: rails
---

If there is a 1-to-1 relation between a web form and a table in the database, everything is nice and easy.

Even if you have a web form which results in creating e.g. a `User` and `Account` record, everything is still nice and easy.

Complexity creeps in when a web form starts having nested, repeatable fields or just a lot of fields.

As an example let's imagine having the concept of a `Document` for which the associated data could be modelled as such:

```json
{
  "title": "Document Title",
  "author_id": 1,
  "parts": [
    {
      "content_type": "text",
      "body": "Just some text in the beginning"
    },
    {
      "content_type": "image",
      "image_url": "http://example.com/some-image.png"
    },
    {
      "content_type": "text",
      "body": "... and some text at the end."
    }
  ]
}
```

This could be modelled as a `Document` model with many `DocumentPart`s.

## The complex method

The approach I would have taken a few years back would be to think that a `Document` cannot exist in a not-done state. This would require having a UI where you can add/remove parts, usually handled with client-side JS. The client-side JS quickly becomes complex because you would have to handle a lot client-side related to the `content_type` parameter in this case.

## The simple method

Instead a lot could be simplified by first letting the user fill in the `title` and then immediately persist a `Document` record. Now we would let the user add the parts one-by-one and this could be simplified even further by making the user select the `content_type` beforehand and presenting a different UI depending on that parameter.

For this method it would probably also make sense to track whether the `Document` is done or being worked on with for instance a `draft` flag. When the user clicks **Save** after building the initial document we would set `draft=false`.

A big win is also that validation becomes very easy because you're no longer validation a complex nested model but usually just a single model at a time.

## Conclusion

Why is the simple method *simpler*? Because once a record is persisted in the database we can leverage the power of having endpoints, e.g. `/documents/1/parts/image/new`. Adding and deleting a `DocumentPart` would also no longer require you to track whether it is a new or existing part as it will be persisted, so you just `DELETE /documents/1/parts/2` if the user wants to delete a part.

One slight disadvantage of "The simple method" is that for API integration you would like to make a single request to create the `Document` + `DocumentPart`s, so reusing controllers could become tricky. Nonetheless "The simple method" has still a lot of advantages.
