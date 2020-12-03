# Week 2 Blog Post

Assignee(s): Emily Chen
Category: Bit Camp
Department: Student Programs
Last Edited: Dec 2, 2020 6:56 PM
Status: In progress

[Parse multipart, Face API (beard)](https://github.com/emsesc/bitcamp-serverless)

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

    Think of them like **pre-written bits of code** that are made for us by other developers. All we have to do is install the package, reference it in our code, and voila! We don't have to write extra code.

    > **Example:** Let's say I want to convert an image to a PDF. I can install [this](https://www.npmjs.com/package/images-to-pdf) package, install it with `npm i images-to-pdf` , and successfully convert my images. *I don't have to write extra code to make the file conversion... I can just "depend" on the npm package*

**Commands to type into your console:**

- Where is my console?

    Click on the "Console" tab in the left panel under "Development Tools".

    ![Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled.png](Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled.png)

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

    During Week 3, we'll be making the POST request from a static web app (HTML page). Making the POST request with Postman is just for testing purposes.

1️⃣ **First** let's define the body of the POST request we received.

```jsx
var body = req.body;
```

If you were to context.log() `body` , you would get the raw body content because the data was sent formatted as `multipart/form-data`:

- Why [multipart/form-data](https://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean)?

    Because the image might be fairly large, we must send it in "multiple parts." We'll also be sending it from an HTML form.

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

We're now going to create a Microsoft Cognitive Services Face API:

1. Log into your Azure portal
2. Press **Create a Resource**
3. Press the **AI + Machine Learning** tab on the left

![Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%201.png](Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%201.png)

Press **Face** and fill out the necessary information

### 🤫 Shhh... Secrets! (process.env[])

There are some secret strings we're going to need in order to communicate with the Face API. This includes the **endpoint** and the **subscription key**.

![Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%202.png](Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%202.png)

Enter into your Face API resource and click on **Keys and Endpoint.** You're going to need KEY 1 and ENDPOINT

Now, head back to the Function App, and we're going to add these values into the Application Settings. Follow [this tutorial](https://docs.microsoft.com/en-us/azure/azure-functions/functions-how-to-use-azure-function-app-settings) to do so.

- Why does it have to be secret?

    Short answer: [it's dangerous](https://www.lockr.io/blog/why-you-need-api-key-security/).

**Naming your secrets**

You can name it whatever you want, but make sure it makes sense. We named it "face_key" and "face_endpoint."

## 🐕 Using Fetch to Make a Request

Let's begin by defining a new async function (`analyzeImage()`) that we're going to call later in the `module.exports()` function. This will **take in the image data we parsed** as a **parameter,** make a request to the **Face API**, and return the **beard data**.

***That's kind of a lot... so let's start!***

1️⃣ Start the function and define our secrets 🔑

```jsx
async function analyzeImage(image){
    const subscriptionKey = process.env['face_key'];
    const endpoint = process.env['face_endpoint'];
```

Remember the secrets we added in the application settings earlier? Now we're assigning them to the variables `subscriptionKey` and `endpoint`. Notice how we access the values with `process.env['name']` .

2️⃣ Defining the URI and request parameters 📎

```jsx
const uriBase = `${endpoint}/face/v1.0/detect`;

    let params = new URLSearchParams({
        'returnFaceAttributes': 'facialHair'
    })
```

- The `uriBase` is the URI of the Face API resource we are going to be making a POST request to.
    - *Refer to the [documentation](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (Request URL) for more details.*
- The `params` are what we're storing the parameters of the request in. We want the Face API to return beard data from the image, so we put `'facialHair'` as the value of `'returnFaceAttributes'` .
    - *Refer to the [documentation](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (Request parameters) for more details.*

3️⃣ Sending the request 📨

```jsx
let resp = await fetch(uriBase + '?' + params.toString(), {
        method: 'POST',
        body: image,
        headers: {
            'Content-Type': 'application/octet-stream',
            'Ocp-Apim-Subscription-Key': subscriptionKey
        }
    })
```

We're now going to use `fetch` to make a **POST** request. Adding all the variables we defined previously in `uriBase + '?' + params.toString()` we get something like this: **[Insert your endpoint]/face/v1.0/detect?returnFaceAttributes=facialHair.**

We send the **image data** (this is the image parameter we will call the function with) in the body and **headers**. `'Content-Type'` is the format our image data is in, and `'Ocp-Apim-Subscription-Key'` contains the **subscription key** of the Face API.

4️⃣ Receiving and returning data 🔢

```jsx
let data = await resp.json();
return data;
```

Now all we have to do is access the beard data from `resp` , which we defined as the response from using fetch to POST, in json format.

**We're done with the `analyzeImage()` function! It returns the beard data we requested using fetch. However, we still have one last step.**

5️⃣ Call the function

```jsx
var analysis = await analyzeImage(image);
```

Head back to the `module.exports()` function because we need to call `analyzeImage()` for it to actually **execute**. **Recall** that we defined image using `var image = parts[0].data;` and got the image data from parsing the raw body. 

Now, we're simply passing the image data into the async function (note the await!) and directing the output to `analysis`.

```jsx
context.res = {
        body: {
            analysis
        }
    }
         
context.done()
```

**To close out the function**, we return `analysis` (the beard data and what was outputted from `analyzeImage()`) in `context.res`. This is what you will see when you successfully make a POST request to our HTTP Trigger. 

🥳 **Here's what your final HTTP Trigger should look like:**

The npm dependencies and `module.exports()`:

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

The awesome `analyzeImage()` function:

```jsx
async function analyzeImage(image){
    const subscriptionKey = process.env['face_key'];
    const endpoint = process.env['face_endpoint'];

    const uriBase = `${endpoint}/face/v1.0/detect`;

    let params = new URLSearchParams({
        'returnFaceAttributes': 'facialHair'
    })

    let resp = await fetch(uriBase + '?' + params.toString(), {
        method: 'POST',
        body: image,
        headers: {
            'Content-Type': 'application/octet-stream',
            'Ocp-Apim-Subscription-Key': subscriptionKey
        }
    })

    let data = await resp.json();

    return data;
}
```

## 🚀Testing: Postman

Nearly there! Now let's just make sure our HTTP Trigger actually works...

**Since our Azure Function will be taking a picture in the request, we are going to be using *Postman* to test it**

You can install it from the [Chrome Store](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en) as a Chrome extension.

*What will Postman do?*

> We are going to use Postman to send a POST request to our Azure Function to test if it works, mimicking what our static website will do.
Our HTTP trigger Azure Function receives an image as an input and outputs beard data!

1. You can choose to sign up or skip and go directly to the app.
2. Close out all the tabs that pop up until you reach **this screen**

    ![https://user-images.githubusercontent.com/69332964/98034295-c46a9380-1de4-11eb-8f8d-ca508f4e04ef.png](https://user-images.githubusercontent.com/69332964/98034295-c46a9380-1de4-11eb-8f8d-ca508f4e04ef.png)

> Now it is time to send a POST request to the HTTP Trigger Function, so using the drop-down arrow, change "GET" to "POST"
The goal? Receive beard data from an inputted image.

1. **Specifying the API Endpoint:** Enter your function URL, which is the API endpoint, into the text box next to POST
- How to get the function url?

    Go to your Function's code and find this:

    ![Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%203.png](Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%203.png)

    Click to copy!

1. **Setting the Header:** Click on "Headers" and enter `content-type` into Key and `multipart/form-data` into Value. 
2. **Adding your beard image:** Click on "Body" and enter `image` into Key and use the dropdown to select "file" in order to upload an image.

![Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%204.png](Week%202%20Blog%20Post%20cb76d2796f744e22a90e76bd20d7d9cd/Untitled%204.png)

3. Now, just click **Send** and get that beard data!

### 🎉 That's the Week 3 Livestream! Reach out to your mentors if you're having trouble.