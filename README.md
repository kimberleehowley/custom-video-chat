# Raise a virtual hand on custom Daily-co video chats 

If you've found your way to this post, you might've run into a similar problem that I did. Maybe you: 

* Teach students in an online classroom, and need a way for them to let you know when they have something to say. 
* Manage public meetings for city supervisors, and want to find an inclusive way to allow for public comment. 
* Just want your family catch-up chats to have a little more order. 

In any of these (and many other!) situations, adding a tool to video chats that lets participants raise their hands, and that lets the moderator see everyone waiting to speak, can come in...handy. 

[Insert groaning gif here]

This post will walk you through how to use the [Daily.co API](https://docs.daily.co/docs/reference-docs) to implement a Raise Your Hand feature on a custom video chat embedded in your website. 

## How it will work 
We'll wire our website up to the [Daily.co API](https://docs.daily.co/docs/reference-docs), which does some under the hood magic to add video chats to any app or website in minutes. We'll add some event listeners to notify us about specific actions (like when a new user joins a call) and respond accordingly, [toggling styles and icons](https://www.w3schools.com/jsref/met_document_getelementbyid.asp) based on those events. 

To set this all up, you'll want to have a Daily-js account (sign up for one [here](https://dashboard.daily.co/)). I'll assume you already have a website up and running that you want to add this to, but, if you don't, you can fork one of [Daily's demos on Github](https://github.com/daily-co/daily-demos), or get some ideas for setting one up over at [Dev.to](https://dev.to/gaelthomas/how-to-deploy-a-static-website-for-free-in-only-3-minutes-with-google-drive-254c).

## How to build it 

### Connect to Daily.co and create a room 
- Add line to head 
- Initialize two variables: boolean to track unique handState, object to collect them all as we go 
- Create the startCall function. -- purposes of getting up quickly, I just wired url. Be more secure than I am. 
- Wire StartCall to body 
- Should see style-less
- I added custom styles using Daily.co's demo to get this up and running quickly 


### Create your event listeners for when a user joins a call 
- Daily.co provides a few events that we can listen for 
- Add functions as callbacks 
- Joins a call: remove loading screen and display leave call option 
- Once in call, can toggle style of hand using getElementById
- But that's just the display. To make that choice visible to everyone, need to send a message 
- After sending message, need to create action for what app does once message is sent 
- Toggles state of list via updateHandState 

[gif about being all alone]

### When other users join a call 
- 'participant-joined' --> newParticipant function 
- .sendAppMessage() is great, but anybody not in the call when message sent, including fashionably late user, won't see it 
- So, when a new user joins, we need to not just add their names to the list with some fun getElementbyId, but also retrigger states from every other user already on the call. 
- Loop through callFrame.participants(), for every user not on the call, send another message
- Which means handState() will be called to run again 

### Saying goodbye 
- Make sure similar hand state adjustment when a user leaves
- Oh and call .destroy() 

### What's next? 
And there you have it! For managing larger applications, with lots of hands raised, recommend using a framework like React. Have a look at [Daily's React demo](https://www.daily.co/blog/building-a-custom-video-chat-app-with-react). And, if you're up for the challenge of getting Daily up on a Gatsby site, give me a shout. 