# 🤚 Raise a virtual hand on custom Daily.co video chats
If you've found your way to this post, you might've run into the same problem I did. Maybe you: 

* Teach students in an online classroom, and need a way for them to let you know when they have something to say without talking over each other. 
* Manage public meetings for city supervisors, and want to find a nondisruptive, organized way to allow for public comments. 
* Just want your family catch-up chats to have a little more order. 

In any of these (and many other!) situations, adding a tool to video chats that lets participants raise their hands, and that lets the moderator see everyone waiting to speak, can come in...handy. 

<div align="center">
<img src="https://media.giphy.com/media/xT0GqtpF1NWd9VbstO/giphy.gif" alt="Cat slapping face"/> 
</div>


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

![](https://gist.github.com/kimberleejohnson/68a947f5043174bc42bd6c02374aa67d)

With that all loaded, we can create a `<script></script>` tag in our body to work with the Daily.co API. I decided to use a tag instead of importing a file to optimize for faster page loading, but you can also import a script if you prefer. 

First, we'll initialize two variables: a boolean to track whether or not a hand is raised `raisingHand`, and an object to pair that status with a user's session id `handState.` 

With those ready globally, we can setup `startCall()`. In this function we'll need to create the DailyIframe, and point it to the appropriate room to join. For demo purposes, I'm hard-coding a url from my [Daily.co dashboard](https://dashboard.daily.co/), but for full production-ready web applications, you'll want to [create a unique room](https://www.daily.co/blog/video-call-api-tutorial-the-rooms-family-of-endpoints). 

After establishing the room link, we call the Daily.co Api to create the iframe, and to put it on our #call-frame element. I added a property to use Daily.co's [custom layout functionality](https://www.daily.co/blog/using-css-grid-to-create-custom-api-video-call-layouts), but that's optional. 

With the callFrame initialized, we then tell our function to join the call with `callFrame.join()`. 

`gist:kimberleejohnson/bd797acb5194da0b458c11599e2c138b#initialize-callframe.html`

With that, you should be able to join a call on loading the page. Now we can add the specifics. 

### React when a user joins a call 
The [Daily.co API](https://docs.daily.co/reference#events) gives us some events we can listen for and react to with custom functions ([callbacks](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)).

We'll start with `joined-meeting`. When a user joins a meeting, we want to: remove their loading screen, and swap the "Join Call" option for "Leave Call". We can do all of that in a `joinedCall()` function we pass to `joined-meeting`. 

`gist:kimberleejohnson/fedf229c9b32bdade5382ed21229892d#joinedCall.html`

### Let a user raise their hand
I added a Raise Your Hand option to my user's controller menu, and connected that to `onClick=toggleHand()` function. 

The function uses `document.getElementById()` to toggle styling based on a user's click, and set that `raisingHand` boolean to match.

But, that click just controls a user's own view. To make sure the change they make is visible to all future callers, we need to call Daily.co's `callFrame.sendAppMessage()` to let everyone else know about the change. 

`gist:kimberleejohnson/ed4ea174790b8653da851866b8c92e4e#toggleHandState.html`

Since [the message won't last forever](https://docs.daily.co/reference#%EF%B8%8F-sendappmessage), we want to take advantage of the `app-message` event listener Daily.co gives us to do something with the message and update the hand list. We do that by manipulating our `handState` object in another callback function. I'm calling this one `updateHandState()`.  

`gist:kimberleejohnson/582b3d56842cebfa15afdf9982e5a980#updateHandState.html`

Now, we're ready for other users to join the call. 

### Update when more users join a call 
The `participant-joined` Daily.co event lets us know when another user joins the call. 

When another user joins us, we need to do two main things: add their name to the guest list, and make sure they can see who else is already on the call and has their hand raised. A new user won't see any messages that were sent before they joined the call, so, when a participant joins, we need to tell every other user to send a new message about whether or not their hands are raised. 

The `updateParticipantList(e)` function takes care of all that. It calls Daily.co's `callFrame.participants()` to get all the participants on the call, loop through them and add their .user_name or the placeholder "Guest" to the list. It also grabs each participant's .session_id to put as an id on the `<img>` tag. For the final step in the loop, for every participant on the call who is not the one who just joined (which we can tell from the event we passed in), we send a message to all the other callers with the appropriate handState, triggering `updateHandState(e)` to run again with the sent message. 

`gist:kimberleejohnson/07ee2c40a725f813af21819cc44d2610#updateParticipantList.html`

### Update when users leave the call 
All good things (and video chats) must come to an end. When a user leaves a call, we need to reset their view to display the "Join Call" option again and remove the "Participants" display, including the names of other users. 

When a user sees another user leave a call through the 'participant-left' event, we need to run `updateParticipantList(e)` again so that their name doesn't remain on the display once they've left.

And, there you have it! Participants can now raise their hands during your video chat. 

`gist:kimberleejohnson/29dca0babf98e7698a04f56242405677#leftCall.html`

And, with that, participants can raise and lower their hands in your custom video chat. 

### Going further
If you're planning on lots of participants in your call, with lots of hands raised, I recommend using a framework like React to manage all those hand statuses. Have a look at [Daily's React demo](https://www.daily.co/blog/building-a-custom-video-chat-app-with-react). And, if you're up for the challenge of getting Daily up on a Gatsby site, please give me a shout over at [@kimeejohnson](https://twitter.com/kimeejohnson). 