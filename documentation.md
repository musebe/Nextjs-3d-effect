### Nextjs pdf splinter


## Introduction

This article shows how the Nextjs framework can be implemented to produce a 3D illusion photo from an image.
## Codesandbox

Check the sandbox demo on  [Codesandbox](/).

<CodeSandbox
title="mergevideos"
id=" "
/>

You can also get the project Github repo using [Github](/).

## Prerequisites

Entry-level javascript and React/Nextjs knowledge.

## Setting Up the Sample Project

In your respective folder, create a new nextjs app using `npx create-next-app 3dimage` in your terminal.
Head to your project root directory `cd 3dimage`
 
Nextjs has its own serverside rendering backend which we will use for our media files upload. 
Set up [Cloudinary](https://cloudinary.com/?ap=em)  for our backend. 

Create your Cloudinary account using this [Link](https://cloudinary.com/console).
Log in to a dashboard containing the environment variable keys necessary for the Cloudinary integration in our project.

Include Cloudinary in your project dependencies: `npm install cloudinary`.

Create a new file named `.env.local` and paste the following guide to fill your environment variables. You locate your variables from the Cloudinary dashboard.

```
CLOUDINARY_CLOUD_NAME =

CLOUDINARY_API_KEY =

CLOUDINARY_API_SECRET =
```

Restart your project: `npm run dev`.

In the `pages/api` directory, create a new file named `upload.js`. 
Start by configuring the environment keys and libraries.

```
var cloudinary = require("cloudinary").v2;

cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
});
```

Create a handler function to execute the POST request. The function will receive media file data and post it to the Cloudinary website. It then captures the media file's Cloudinary link and sends it back as a response.

```
export default async function handler(req, res) {
    if (req.method === "POST") {
        let url = ""
        try {
            let fileStr = req.body.data;
            const uploadedResponse = await cloudinary.uploader.upload_large(
                fileStr,
                {
                    resource_type: "video",
                    chunk_size: 6000000,
                }
            );
            url = uploadedResponse.url
        } catch (error) {
            res.status(500).json({ error: "Something wrong" });
        }

        res.status(200).json({data: url});
    }
}
```
The code above concludes our backend.

We'll use [styled-components](https://www.npmjs.com/package/styled-components) to create a card that can rotate 180 degrees. Include it in the dependencies:

`npm install styled-components` 
We use the following code to create our card component. We first show the front and hide the back card. The back will listen to the `flip` state hook to rotate 180 degrees to reveal the back.

```
"pages/index"

import styled from "styled-components";
import { useState } from "react";

const Container = styled("div")(() => ({
  display: "flex",
  flexDirection: "row",
  justifyContent: "center"
}));

const Card = styled.div`
  position: relative;
  flex-basis: 100%;
  max-width: 220px;
`;

const CardTemplate = styled("div")(() => ({
  width: "100%",
  backfaceVisibility: "hidden",
  height: "400px",
  borderRadius: "6px",
  transformStyle: "preserve-3d",
  transition: "transform 1s cubic-bezier(0.8, 0.3, 0.3, 1)"
}));

const CardFront = styled(CardTemplate)(({ flip }) => ({
  backgroundImage: "url('images/cardimage.jpg')",
  backgroundSize: "cover",
  backgroundPosition: "center",
  transform: flip ? "rotateY(-180deg)" : "rotateY(0deg)"
}));

const CardBack = styled(CardTemplate)(({ flip }) => ({
  position: "absolute",
  top: 0,
  left: 0,
  background: "linear-gradient(0deg, #00adb5 20%,#596a72 100%)",
  transform: flip ? "rotateY(0deg)" : "rotateY(180deg)"
}));

const CardContent = styled("div")(() => ({
  top: "50%",
  position: "absolute",
  left: 0,
  width: "100%",
  backfaceVisibility: "hidden",
  transform: "translateZ(70px) scale(0.90)"
}));

const BGFade = styled("div")(() => ({
  position: "absolute",
  right: 0,
  bottom: 0,
  left: 0,
  height: "200px",
  background: "linear-gradient(to bottom, rgba(0,0,0,0) 20%,rgba(0,0,0,.8) 90%)"
}));

export default function Home() {
  const [flip, setFlip] = useState(false);

  return (
    <div className="App">
 
    </div>
  )
}
```
Finally, paste the following in your return statement:

```
return (
    <div className="App">
      <h1>Nextjs 3d Photo Effect</h1>
      <Container>
        <Card
          onMouseEnter={() => setFlip(true)}
          onMouseLeave={() => setFlip(false)}
        >
          <CardFront flip={flip}>
            <CardContent>
              <h1>Front</h1>
            </CardContent>
            <BGFade />
          </CardFront>
          <CardBack flip={flip}>
            <CardContent>
              <h1>Back</h1>
            </CardContent>
          </CardBack>
        </Card>
      </Container>
    </div>
  )
```
That's it! Your UI should look like the below:

![complete UI](https://res.cloudinary.com/dogjmmett/image/upload/v1654775281/UI_jseh11.png "complete UI")

Use the following code to command your upload button. The code will be uploaded to the backend for Cloudinary upload.

```
  const uploadHandler = async () => {
    console.log(cardRef.current)
    await html2canvas(cardRef.current).then(function (canvas) {
      try {
        fetch('/api/upload', {
          method: 'POST',
          body: JSON.stringify({ data: canvas.toDataURL() }),
          headers: { 'Content-Type': 'application/json' },
        })
          .then((response) => response.json())
          .then((data) => {
            console.log(data.data);
            setSampleSelected(true)
          });
      } catch (error) {
        console.error(error);
      }
    })
  }
```

That accomplishes the project. Enjoy selecting pictures for your card. An example is in the front card.