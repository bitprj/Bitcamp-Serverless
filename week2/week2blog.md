## **Week 2**

Last week, you should've learned the basics of how to create an Azure Function, along with the basics of triggers and bindings.

### **Learning Objectives**

- retrieving data from Face API
- making HTTP requests using fetch
- installing and using npm dependencies

### **Livestream**

In the livestream, we're going to code a HTTP trigger Azure Function that detects facial hair in a submitted picture.

- For the full video, look in the [video folder](https://github.com/emsesc/bitcamp-serverless/blob/master/week2/livestream/videos).
- For the full code, look in the [code folder](https://github.com/emsesc/bitcamp-serverless/blob/master/week2/livestream/code).

**We'll be going over how to:**

1. configure npm dependencies in Functions
2. parse multipart form data
3. create a Face API resource
4. make a HTTP request to the Face API
5. test the function using Postman

---

## 📝 Review: Creating the HTTP Trigger

Like last week, we'll be creating an HTTP Trigger to parse the image and analyze it for beard data! 🧔

- We're going to need a separate **Function App** for this project, so let's [create a new one](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function#create-a-function-app).

> Tip: It might be helpful to keep track of the Function App name and Resource Group for later in the project.

- Now that we have a new Function App, [create a new **HTTP Trigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function#create-function)** to begin the actual coding.

## 🖥️ Installing Dependencies

We are going to be using some npm packages in our HTTP Trigger, so we must install them in order for our code to even work.

- What are npm packages/dependencies?

**Commands to type into your console:**

- Where is my console?

`npm init -y`

`[npm install parse-multipart](https://www.npmjs.com/package/parse-multipart)`

`[npm install node-fetch](https://www.npmjs.com/package/node-fetch)`

**Last step:**

Be sure to add these initializing statements into the code:

```jsx
var multipart = require('parse-multipart');
var fetch = require('node-fetch');
```

This defines `multipart` and `fetch` , which we will use soon in the code below. ⬇️

## 🖼️ Parse Multipart

The first step is to navigate to the new HTTP trigger you created earlier. You should see this at the top of the default code:

```jsx
module.exports = async function (context, req) {
	context.log('JavaScript HTTP trigger function processed a request.');
```

We're going to be working in this function!

The **parse-multipart library** (`multipart`) is going to be used to parse the image from the POST request we will later make with Postman to test. 

- Sidenote:

1️⃣ **First** let's define the body of the POST request we received.

```jsx
var body = req.body;
```

If you were to context.log() `body` , you would get the raw body content because the data was sent formatted as `multipart/form-data`:

- Why [multipart/form-data](https://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean)?

```yaml
------WebKitFormBoundaryDtbT5UpPj83kllfw
Content-Disposition: form-data; name="uploads[]"; filename="somebinary.dat"
Content-Type: application/octet-stream

some binary data...maybe the bits of a image.. (this is what we want!)
------WebKitFormBoundaryDtbT5UpPj83kllfw
```

2️⃣ **Secondly,** we need to create the boundary string from the headers in the request.

```jsx
var boundary = multipart.getBoundary(req.headers['content-type']);
```

If you were to context.log() `boundary` , you would receive `--WebkitFormBoundary[insert gibberish]` , which is a boundary that helps you determine where the image data is. This boundary will help us **parse** out the image data. Take a look at the boundaries in the raw payload above in step 1 ⬆️

3️⃣ **Third!** Now we'll be using `multipart` again to actually parse the image. 

```jsx
var parts = multipart.Parse(body,boundary);
```

This returns an **array** of inputs, which contains all the different files that were in the body. In our case, we only have one: the picture!

- *Each object in the array as the attributes of filename, type, and data (we want the data)*

4️⃣ **Finally.** We can now access the image with this short and sweet line of code:

```jsx
var image = parts[0].data;
```

The array "image" only has one object, the picture, so we've now **successfully parsed the image** from the body payload 🎉

**Here's what your code should look like now:**

```jsx
var multipart = require('parse-multipart');
var fetch = require('node-fetch');

module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
  
    // receiving an image from an html form
  
    //enctype = multipart/form-data
    //parse-multipart library 

    var body = req.body;

    //returns ---WebkitFormBoundaryjf;ldjfdlf
    var boundary = multipart.getBoundary(req.headers['content-type']);

    //returns an array of inputs
    //each object has filename, type, data
    var parts = multipart.Parse(body,boundary);

    //array has 1 object(an image)
    var image = parts[0].data;
}
```

## 🙃 The Face API

## 🐕 Using Fetch to Make a Request

Here's what your final HTTP Trigger should look like:

```jsx
var multipart = require('parse-multipart');
var fetch = require('node-fetch');

module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
  
    // receiving an image from an html form
  
    //enctype = multipart/form-data
    //parse-multipart library 

    var body = req.body;

    //returns ---WebkitFormBoundaryjf;ldjfdlf
    var boundary = multipart.getBoundary(req.headers['content-type']);

    //returns an array of inputs
    //each object has filename, type, data
    var parts = multipart.Parse(body,boundary);

    //array has 1 object(an image)
    var image = parts[0].data;

    var analysis = await analyzeImage(image);

    context.res = {
        body: {
            analysis
        }
    }
         
    context.done()
}
```

## 🚀Testing: Postman
