# HTTP Conventions

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a>

> This is part of Academy's [technical curriculum for **The Mark**](https://github.com/WeAreAcademy/curriculum-mark). All parts of that curriculum, including this project, are licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.

Now you've built a working server, we'll start adding some common conventions on top of them - focusing on status codes and request types.

We'll explore this through a server which performs **CRUD operations** on a data store of signatures - like you might get in a guest book in a museum, perhaps.

## Learning Outcomes

- Use conventions on HTTP request types
- Use conventions on HTTP status codes
- Use a route parameter in Express
- Use JSend formatting
- Send JSON body data with Postman
- Handle JSON body data with Express (including middleware)

Make sure you do the reading in Exercise 0 _before_ you look at this project code - parts of it will be very cryptic without it.

## Exercise 0: Conventions in request types and status codes

### HTTP request types

It is common to think about CRUD actions when interacting with a server (and, later, a database):

- **C**reate
- **R**ead
- **U**pdate
- **D**elete

for example:

- hatching a digipet is _creating_ something (a 'resource')
- viewing a digipet's stats is _reading_ a resource
- feeding your digipet is _updating_ a resource
- rehoming your digipet is _deleting_ a resource

We've been exclusively making `GET` requests and defining corresponding `app.get` route handlers - instead, we're going to start pairing up the intended CRUD action to the commonly associated HTTP request type:

| Action | Request type | Example |
| --- | --- | --- |
| Create | POST | POST `/todos` to create a new todo |
| Read | GET | GET `/todos` to view all todos; GET `/todos/4` to get todo #4 |
| Update | PUT\* | PUT `/todos/4` to update todo #4 |
| Delete | DELETE | DELETE `/todos/4` to delete todo #4 |

\*either a PUT or a PATCH can be used to update. They have slightly different semantics, but we'll set those aside for now and exclusively use PUT for the sake of simplicity.

More reading: [https://restfulapi.net/http-methods/](HTTP methods)

### HTTP status codes

When we send a response from our server, it has a numerical status code attached.

In Express, this defaults to `200` - but [there are lots more to choose from](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

We don't need to learn all of these, but here are some ones that are more common

- [`200` (OK)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200)
- [`201` (Created)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/201)
- [`400` (Bad Request)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400)
- [`404` (Not Found)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404) - the classic

And, one which is pretty rare:

- [`418` (I'm a teapot)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/418) - yes, really

Most of the time, we will be using `200` (there's a good reason for it to be the Express default), but we will be more explicit about this and occasionally dip into other status codes.

### JSON response structure: JSend

We could send back our JSON data in any particular structure we want.

For example, in response to a `GET /todos`, here are some possible JSON responses:

```json
[{ "text": "Buy some milk" }, { "text": "Solve the P versus NP problem" }]
```

```json
{
  "message": "Showing all todos",
  "data": [
    { "text": "Buy some milk" },
    { "text": "Solve the P versus NP problem" }
  ]
}
```

```json
{
  "report": "Successfully fetched 2 todos",
  "metadata": {
    "time": 104
  },
  "fetched": {
    "todos": [
      { "text": "Buy some milk" },
      { "text": "Solve the P versus NP problem" }
    ]
  }
}
```

```json
{
  "errors": [],
  "data": {
    "resource": "todos",
    "content": [
      { "text": "Buy some milk" },
      { "text": "Solve the P versus NP problem" }
    ]
  }
}
```

Some formatting guidelines and conventions have emerged, with the most popular being a specification called [JSON:API](https://jsonapi.org/) - but it's pretty meaty and overkill for the small APIs which we're building...!

Instead, for learning purposes, we'll use the simpler (if less popular) [JSend specification](https://github.com/omniti-labs/jsend), which would prescribe:

```json
{
  "status": "success",
  "data": {
    "todos": [
      { "text": "Buy some milk" },
      { "text": "Solve the P versus NP problem" }
    ]
  }
}
```

### Conventions vs requirements

You don't _have_ to use any of these conventions - it is perfectly possible to write a server which performs CRUD operations without using conventional HTTP request types, status codes or JSON response specifications.

(This is exactly what we did with our digipet backend - exclusively GET requests, and exclusively `200` status codes by default.)

However, writing our server by following conventions will make it much easier for others to work with it.

## Exercise 1: Focused code reading, and making your first POST request with data

> 🎯 **Success criterion:** you are able to **C**reate new signatures on your server and **R**ead back all signatures

As a software engineer, you will be working on codebases which are large, have been written by other people and where most of it may be code that you never touch - you work on a relatively small surface area of it.

This project is a mini exposure to the same idea. Explicitly: we don't expect that you'll have to work on any of the code in the `signature` directory (you may have to _use_ a couple of the functions within, but you needn't understand all of it). If you would like, you can treat it entirely as a black box. (You're welcome to poke around in it if you want to look, but you're not required to and it's not important for the main learning outcomes of this exercise.)

We'll be working on the `server.ts` code and, eventually, unit tests.

### `server.ts`

Read through the `server.ts`, particularly with the lens of HTTP methods, status codes and JSON response specification.

You'll see that, even though `200` is the default status code, there are lots of times in the code that it is _explicitly set_ as `200` - this is to communicate that it is a _deliberate choice_ of 200, rather than simply an accident of defaulting to it.

### Sending body data via Postman

We're now going to test the server's support for **C**reating and **D**eleting signatures.

Clone/fork the repo and start it locally.

Open up Postman and make a GET request to `localhost:4000/signatures`. You should see that the request is made and handled successfully - but that there are, as of yet, zero signatures (i.e. the signatures array is empty).

As we've learned, a POST request is the conventional way to create a signature - so let's try that.

Open up a new request tab (the + near the top) and create a request to `localhost:4000` - but this time, switch it to a POST request - and then submit it.

You'll get a response like this:

```json
{
  "status": "fail",
  "data": {
    "name": "A string value for name is required in your JSON body"
  }
}
```

When we made a `POST /signatures` request, we asked our server to create a new signature - but it can't tell which signature it should be creating.

We're modelling signatures as objects having a `name` property which is a `string` (take a quick peek at the `Signature` interface in `signature/model.ts`) - so we should, somehow, provide information to our server on what `name` it should be using when creating this new signature.

Let's do this by, just underneath the request URL in Postman, clicking onto the 'Body' tab. A request body lets us add extra information into our request (just like a server response body has more information about the response).

Click on the 'raw' option, and change the dropdown on the right to JSON, to say that we're sending some JSON data, and then type into the text area:

```json
{
  "name": "Cicely Popplewell"
}
```

so that it looks like the below:

<!-- TODO: add image here -->

Now, when you send that request, you'll get back something like:

```json
{
  "status": "success",
  "data": {
    "signature": {
      "name": "Cicely Popplewell",
      "epochId": 1614605810324
    }
  }
}
```
which _seems_ to indicate that we have successfully added a signature in. (Your `epochId` will be different - it's related to creation time.)

Now, change back to our tab where we're making a GET request to `/signatures`, and resend it - you'll see it's now fetching back an array of data which includes the signature we just created!

Now, repeat this to show that you can add 2-3 (different) signatures which are all retrieved when we make subsequent GET requests to `/signatures`.

### Understanding the behaviour

Look at the code implemented in the route handlers for GET `/signatures` and POST `/signatures` to see if you can make sense of the above behaviour which we can see.


## Exercise 2: Route parameters, and making your first DELETE request

> 🎯 **Success criterion:** you are able to **R**ead and **D**elete specific signatures from the server via Postman

### Reading a specific signature using a route parameter

In `server.ts`, we have a GET route handler for the route `/signatures/:epoch`. This is using a route parameter (with documentation linked to in the comments).

Let's try this out. Make a GET request to `/signatures` and pick one of the `epochId`s which you see - for example, I might pick `1614605810324` for Cicely Popplewell. 

Then, create a new tab, and make a GET request of the structure `/signatures/1614605810324` (substituting in your chosen `epochId`).

You should see that the matching signature comes back!

Experiment with changing it to other `epochId`s - including ones which aren't present in your array of signatures.

Then, return to the GET route handlers for `/signatures/:epoch`. How does this code relate to the behaviour you've observed?

### Making a DELETE request

See if you can figure out from the above how to now _delete_ specific signatures.

It's very similar...

## Exercise 3: Reading the tests

> 🎯 **Success criterion:** you are able to explain how we unit test our server response

We're now going to look at the `server.test.ts` file and see how we're testing the above behaviour in some way.

### A note on mocking

One of the things you'll see in the test file is some test code which references 'mocking' - so, before you go into the file, we'll take a moment to step back and think about mocking.

Think back to the unit tests in Digipet - we saw that we separated out our controller unit tests and our server unit tests such that it was possible for one to fail without the other failing.

We achieved this through the use of mocks, which let us test whether a (mocked version of a) function gets called, without tying us into the specific behaviour of a function.

In `server.test.ts`, you'll see some code that looks like this under the tests for `GET /signatures` (and similar code in other `describe` blocks):

```ts
beforeEach(() => {
  resetMockFor(getAllSignatures, () => [...mockResponseData]);
});
```

This is using:
1. A Jest-provided utility, `beforeEach`, which is [used for some test setup](https://jestjs.io/docs/en/setup-teardown); and
2. A custom test utility, `resetMockFor` (written in `test-utils.ts`)

The exact implementation and details is not important - if you want, you can ignore all `beforeEach` blocks - but the picture here is that we tell Jest that:
1. In our test for `GET /signatures`, we want to provide a mock for `getAllSignatures` - making this a nicely isolated server unit test that isn't dependent on the _actual_ implementation of `getAllSignatures`
2. What's more, where Jest sees any calls to `getAllSignatures`, it should fill in the behaviour using the anonymous callback function `() => [...mockResponseData]`

This lets us, e.g., test that the server response to `GET /signatures` includes the result of `getAllSignatures` - but swapping in a mocked and simplified version of `getAllSignatures` to use in place of the real function. It lets us test how the endpoint handler _uses_ `getAllSignatures` without worrying about the exact details of how it's implemented in reality.

You don't need to worry about the exact specifics of what's being mocked as much as the _intent_ - the job of these mock functions is to do a plausible job of returning what the actual function _might_ return, without having to do any gory implementation details.

### Reading the test code

Now, try having a look at the test code and making sense of it.

(The tests for PUT are being skipped right now - we'll be writing a handler for that soon.)

In particular, look for the following:
- How are status codes being tested?
- How is sending data in a JSON body being simulated and tested?


## Exercise 4: Testing `DELETE /signature/:epoch`

> 🎯 **Success criterion:** you have a test written for DELETE requests made to `/signature/:epoch`

Write a test for the DELETE `/signature/:epoch` route handler.

It will look very similar to some other tests (although it's up to you to figure out how!).

Once you've written it, so that it passes, you should also **check that it is possible for the tests to fail** by deliberately modifying the server code in a way which should make the tests fail. (Tests that pass no matter what aren't helpful - we want tests which tell us when we're doing something wrong.)

When the tests pass, check that you can perform **D**elete actions as expected on signatures.

## Exercise 5: Writing `PUT /signature/:epoch`

> 🎯 **Success criterion:** you have a route handler written for PUT requests made to `/signature/:epoch` that passes the tests

Now, unskip the `describe` block for the tests for PUT `/signature/:epoch` - you'll find that they fail when you run the tests.

It's up to you to write the route handler to make the tests pass!

After that, see if you can perform **U**pdate actions as expected on signatures.

## Exercise 6: Writing integration tests

We've now got unit tests for CRUD actions against the `/signatures` endpoint - we've tested that we're getting back sensible server status code and response body structures, and that the relevant model functions are being called.

We don't have any integration tests though - it'd be good if we could check that these all fit together in some expected way, i.e. along the lines of:
1. Firstly, `GET /signatures` responds with an empty array for signatures
2. After making a `POST /signatures`, there's now a signature in the array
3. We can also now make a `GET /signatures/:epoch` request using the `epochId` of the newly created signature
4. We can update the same signature using `PUT /signatures/:epoch`
5. When I read the signature again through `GET /signatures/:epoch` I can see that it has indeed updated
6. I can delete the signature via `DELETE /signatures/:epoch`
7. Now, when I try to `GET /signatures/:epoch` for that signature, it returns a 404
8. Reading all signatures at `GET /signatures`, I can see the signature has been removed



