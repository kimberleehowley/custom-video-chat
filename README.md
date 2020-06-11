 # Raise a virtual hand on custom video chats with Daily-co

If you've found your way to this post, you might've run into a similar problem that I did. Maybe you: 

* Teach students in an online classroom, and need a way for them to let you know when they have something to say. 
* Manage public meetings for city supervisors, and want to find an inclusive way to allow for public comment. 
* Just want your family catch-up chats to have a little more order. 

In any of these (and many other!) situations, adding a tool to video chats that lets participants raise their hands, and lets the moderator see everyone waiting to speak, can come in...handy. 

[Insert groaning gif here]

This post will walk you through how to use the [Daily-co JS]() API to implement this kind of feature on a custom video chat embedded in your website. 

## How it will work 
We'll wire our website up to the Daily.co API, which does some under the hood magic to add  

To set this all up, you'll want to have a Daily-js account (sign up for one [here]()). I'll assume you already have a website up and running that you want to add this to, but, if you don't, you can fork one of [Daily's demos on Github](https://github.com/daily-co/daily-demos), or get a refresh on the basics over on [Free Code Camp](https://www.freecodecamp.org/news/how-to-deploy-a-static-website-for-free-in-only-3-minutes-with-google-drive/).

## How to build it 