# 🤚 Raise a virtual hand on custom Daily.co video chats
If you've found your way to this post, you might've run into the same problem I did. Maybe you: 

* Teach students in an online classroom, and need a way for them to let you know when they have something to say without talking over each other. 
* Manage public meetings for city supervisors, and want to find a nondisruptive, organized way to allow for public comments. 
* Just want your family catch-up chats to have a little more order. 

In any of these (and many other!) situations, adding a tool to video chats that lets participants raise their hands, and that lets the moderator see everyone waiting to speak, can come in...handy. 

<div align="center">
<img src="https://media.giphy.com/media/xT0GqtpF1NWd9VbstO/giphy.gif" alt="Cat slapping face"/> 
</div>
</br>

This post will walk you through how to use the [Daily.co API](https://docs.daily.co/docs/reference-docs) to implement a Raise Your Hand feature on a custom video chat embedded in your website. 
## How this will work 
We'll wire a website up to the [Daily.co API](https://docs.daily.co/docs/reference-docs), which does some under the hood magic to add video chats to any app or site in minutes. We'll listen for specific Daily.co call events (like when a new user joins) and execute custom callback functions in response. 

To set this all up, you'll want to have a Daily-js account (sign up for one [here](https://dashboard.daily.co/)). I'll assume you already have a website in mind to add this feature to, but, if you don't, you can fork one of [Daily.co's demos on Github](https://github.com/daily-co/daily-demos), or get some ideas for building a website over on [Dev.to](https://dev.to/gaelthomas/how-to-deploy-a-static-website-for-free-in-only-3-minutes-with-google-drive-254c).

If you'd rather head straight to the code, you can check out the [full demo repository](https://github.com/kimberleejohnson/custom-video-call). 

Or, if you'd rather see the demo live in action, head over to [Netlify](https://daily-chat-raise-your-hand.netlify.app/). 

![Screenshot of the site in action](./icon-assets/daily-demo-cropped.png)

## How to build this 
### Connect to Daily.co 
In the head of the html page where you'll host the call, add the @daily-co script tag. 

Next, inside the `<body>` tag, add an onload function to start the call as soon as someone visits the page: `onload="startCall()"`. We'll write that function soom, but first add the iframe where the Daily.co call will load within the body of the page, making sure it has an id tag. The top of your file should now look something like this: 

```html 
<html>
  <head>
    <title>Video chat</title>
    <script src="https://unpkg.com/@daily-co/daily-js"></script>
    <link rel="stylesheet" href="styles/page-styles.css" />
  </head>
  <body onload="startCall()">
    <iframe id="call-frame" allow="camera; microphone; autoplay"></iframe>
    <!-- More to come here, including our script --> 
  </body>
</html> 
```

With that all set, we can create a `<script></script>` tag in our body to work with the Daily.co API. A tag within my html file worked faster for me locally than importing a separating .js script, but you could go that route too. 

First, we'll initialize two variables: a boolean to track whether or not a hand is raised `raisingHand`, and an object to pair that status with a user's session_id `handState.` 

With those ready, we can setup `startCall()`. In this function, we'll need to create the DailyIframe, and point it to the appropriate room to join. I hard-coded a url from my [Daily.co dashboard](https://dashboard.daily.co/) to get this demo running quickly, but for full production-ready chats, you'll want to [create a unique room](https://www.daily.co/blog/video-call-api-tutorial-the-rooms-family-of-endpoints). 

We can now call the Daily.co API to create the iframe within our #call-frame element (that's why we needed an id). I added a property to use Daily.co's [custom layout functionality](https://www.daily.co/blog/using-css-grid-to-create-custom-api-video-call-layouts) too, but that's optional. 

With the callFrame initialized, we then tell our function to join the call with `callFrame.join()`. 

```html 
<html>
  <!--  -->
  <script>
    let raisingHand;
    let handState = {};
    
    async function startCall() {
    // For demo purposes, I'm hard-coding the meeting room.
    // This is NOT a best practice in production. 
    // For more on creating rooms securely, head to: https://docs.daily.co/reference#rooms
      room = { url: "https://your-account.daily.co/hello" };
  
      callFrame = window.DailyIframe.wrap(
        document.getElementById("call-frame"),
          { customLayout: true }
      );
      // Our event listeners will go here 
      await callFrame.join({
          url: room.url,
          cssFile: "./styles/callframe-styles.css",
        });
    }
  // More to come here!
  </script>
  <!--  -->
</html>
```

You should be able to now refresh the page and enter a call. Now to customizing. 

### React when a user joins a call 
The [Daily.co API](https://docs.daily.co/reference#events) gives us some events we can listen for and react to with custom functions ([callbacks](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)).

We'll start with `joined-meeting`. When a user joins a meeting, we want to remove their loading screen, and to swap the "Join Call" option for "Leave Call". We can do all of that in a `joinedCall()` function we pass to `joined-meeting`. 

```html
<html>
  <!--  -->
  <script>
    // ...Our variables are initialized here 
    
    async function startCall() {
    // ...Here we set up our iFrame and joined 
      callFrame
          .on("joined-meeting", joinedCall)
          // More event listeners will be added. 
    // ...Here we joined the call 
    }
    
    async function joinedCall() {
        // Remove the loading screen, display video 
        document.getElementById("ui-local").style.display = "none";
        // Display the Leave Call option, instead of Join Call
        document.getElementById("leave-call-label").innerHTML = "Leave call";
        document.getElementById("leave-call-div").onclick = () => {
          callFrame.leave();
        };
      }
  // More to come here!
  </script>
  <!--  -->
</html>
```

### Let a user raise their hand
I added a Raise Your Hand option to my user's controller list, alongside the microphone and video icons, and connected that hand to an `onclick=toggleHand()` function. 

The function uses `document.getElementById()` to toggle styling based on a user's click, and to then set the `raisingHand` boolean based on whether the hand is raised.

But, that click just controls a user's _own_, or "local" view. To make sure the change they make is visible to all future callers, we need to call Daily.co's `callFrame.sendAppMessage()` to let everyone else know about the change. 

```html
<html>
  <!--  head tag here -->
  <body onload="startCall()">
    <!--  iframe and other elements here -->
    <div onclick="toggleHand()" class="ui-controller-control">
      <p id="handraise-label">Raise hand</p>
      <img src="./icon-assets/icon-raised-hand.png" alt="Hand icon" />
    </div>
    <!--  Additional html elements here -->
  <script>
    // ...Our variables, callFrame, and other functions are here 
    async function toggleHand() {
        if (!raisingHand) {
          raisingHand = true;
          // Change the user's display
          document.getElementById("handraise-label").innerHTML = "Lower hand";
          document.getElementById("local-hand").style.display = "block";
        } else {
          raisingHand = false;
          document.getElementById("handraise-label").innerHTML = "Raise hand";
          document.getElementById("local-hand").style.display = "none";
        }
        // Send a message sharing this state with all callers
        update = {
          status: raisingHand,
        };
        callFrame.sendAppMessage(update, "*");
      }
  // More to come here!
  </script>
  <!--  -->
</html>
```

To do something with that information, we'll use the Daily.co `app-message` event, adding a callback function to update our `handState`. I'm calling this one `updateHandState(e)`.  

```html
<html>
  <!--  -->
  <script>
    // ...Our variables are initialized here 
    
    async function startCall() {
    // ...Here we set up our iFrame
      callFrame
          // Other event listeners here 
          .on("app-message", updateHandState)
          // Other event listeners here 
    // ...Here we joined the call 
    }
    
    async function updateHandState(e) {
        let id = e.fromId;
        let status = e.data.status;
        handState[id] = status;
        if (handState[id] === true) {
          document.getElementById(`${id}`).style.display = "block";
        }
      }
  // More to come here!
  </script>
  <!--  -->
</html>
``` 

### React when other users join the call 
The `participant-joined` Daily.co event lets us know when another user joins the call. 

We need to do two main things when that happens: add their name to the guest list, and make sure they can see who else is already on the call and has their hand raised. A new user [won't see any messages that were sent before they joined the call](https://docs.daily.co/reference#%EF%B8%8F-sendappmessage). So, when a participant joins, we need to tell every other user on the call to send another message to everyone else about whether or not their hand is raised. 

Our `updateParticipantList(e)` calls Daily.co's `callFrame.participants()` to get all the participants on the call, loop through them and add their .user_name or the placeholder "Guest" to the participant list. It also grabs each participant's .session_id to put as an id on the `<img>` tag. As the last step in the loop, each participant on the call who is not the one who just joined (we get that information from the event `(e)` we passed in), sends a message to all the other callers with their handState. Since a message has been sent, our `updateHandState(e)` runs again. 

```html
<html>
  <!--  -->
  <script>
    // ...Our variables are initialized here 
    
    async function startCall() {
    // ...Here we set up our iFrame
      callFrame
          // Other event listeners here 
          .on("participant-joined", updateParticipantList)
          // Other event listeners here 
    // ...Here we joined the call 
    }
    
    async function updateParticipantList(e) {
        // Get all the participants on the call 
        participants = callFrame.participants();
        let wrapper = document.getElementById("ui-participant-guests");
        wrapper.innerHTML = "";
        
        // Loop through participants, adding their names to the list
        Object.keys(participants).forEach((p) => {
          if (p === "local") {
            return; // We exit out here because "You" is already added for the local user
          }
          let participant = participants[p];
          wrapper.innerHTML += `
        <div class="ui-participant-guest">
            <p>${participant.user_name || "Guest"}</p>
            <img id=${
              participant.session_id
            } src="./icon-assets/icon-raised-hand.png" alt="Raised hand" style="display:none"/>
        </div>`;

          // For every participant on the call that is not the one who just joined or left 
          if (p !== e.participant.session_id) {
            // Send everyone a message about the participant's hand state
            update = {
              status: handState[participant.session_id],
            };
            setTimeout(() => {
              callFrame.sendAppMessage(update, "*");
            }, 3000);
          }
        });
      }
  // More to come here!
  </script>
  <!--  -->
</html>
```

### React when users leave the call 
All good things (and video chats) must come to an end. When a user leaves a call, we need to reset their view to display the "Join Call" option, and to remove the list of participants. 

When a user sees another caller leave (on the `participant-left` event), we need to run `updateParticipantList(e)` again so that their name and handState gets removed from the list when they leave. 

```html
<html>
  <!--  -->
  <script>
    // ...Our variables are initialized here 
    async function startCall() {
    // ...Here we set up our iframe
      callFrame
          // Other event listeners here 
          .on("left-meeting", leftCall)
          .on("participant-left", updateParticipantList)
          // Other event listeners here 
    // ...Here we joined the call 
    }
    
    async function leftCall(e) {
        // Display the Join Call option
        document.getElementById("leave-call-label").innerHTML = "Join call";
        document.getElementById("leave-call-div").onclick = () => {
          callFrame.join();
        };
        // Clear the participants' list locally
        document.getElementById("ui-participant-local").style.display = "none"; 
        updateParticipantList(e)
      }
  </script>
  <!--  -->
</html>
```

And, there you have it! Participants can now raise their hands during your video chat. 

<div align="center">
<img src="https://media.giphy.com/media/3o7aTucH1E8PTkEe7S/giphy.gif" alt="High School girls raising hands"/> 
</div>

### Going further
If you're planning on lots of participants in your call, with lots of hands raised, I recommend using a framework like React to manage all that state. Have a look at [Daily's React demo](https://www.daily.co/blog/building-a-custom-video-chat-app-with-react). And, if you're up for the challenge of getting a Daily.co video chat up on a Gatsby site, please give me a shout at `hello@kimberlee.dev`.  