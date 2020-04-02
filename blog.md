![image](https://user-images.githubusercontent.com/3941856/78070976-84749700-7351-11ea-9ada-e1e389c6200a.png)

## Why have the raise your hand feature?

Being able to raise your hand is a "handy" feature to have during video calls. On larger calls, it's almost a requirement if you plan to have interactivity between a host and participants. Here are a few examples taken from my own experiences of the "raise hand" feature in action.
### During a class session or fireside chat with an expert
- There was limited time for Q&A. If students had questions to address to the speaker they would raise their hand and unmute when called on.
### In webinars that are attended by many with one or a few moderators
-  In these calls, everyone except for the moderator was muted with cameras off. Raising a hand was the only way to get the moderator's attention. Chat moves too quickly to monitor.
-----
If you are hosting a call with many in the audience and you want some way for them to quietly grab your attention, you probably need the raise hand feature in your video conferencing app. 

Raise your hand isn't a feature that comes out-of-the-box with the Daily.co pre-built call interface. Luckily, the [Daily-js front-end library](https://docs.daily.co/reference#using-the-dailyco-front-end-library)  exists and you can build out this feature with JavaScript, HTML, and CSS. No backend required.

## Building the sample app with HTML, CSS, and JavaScript

What you'll need to complete this project:
- A text editor
- A [Daily.co](https://daily.co) room URL. You can grab or create one from a free account.

> TL;DR? Check out the [forkable](https://repl.it/@prophen/NavyUniformRoute), [working demo](https://NavyUniformRoute--prophen.repl.co) with source code on Repl.it

This blog post is focused on implementing the raise your hand features with a custom app and the Daily.co prebuilt call interface dropped in.

### HTML structure
#### Include daily-js in `<head>`
I added [Bulma](https://bulma.io/) for easy styling and I put the custom code in its own .css and .js files.
```HTML
  <head>
    ...
    <script defer src="./script.js"></script>
    <script src="https://unpkg.com/@daily-co/daily-js"></script>
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/bulma@0.8.0/css/bulma.min.css"
    />
    <link rel="stylesheet" href="./style.css" type="text/css" />
  </head>
```
The important line of code here for our raised hand feature is: 
```HTML 
<script src="https://unpkg.com/@daily-co/daily-js"></script>`
```
The daily-js library will do some heavy lifting for us. (Thanks Daily ❤️)

#### Create the call UI
Three UI elements are needed demonstrate the Raise Your Hand feature in our sample app.

- The Join Call button
- The list of participants
- The Daily.co call frame


1. Join Call UI
```HTML
          <div class=" welcome-box box ">
            <p class="subtitle">
              Click the button below to join the call
            </p>
            <button
              class="button is-fullwidth is-link is-medium"
              onclick="run()"
            >
              Join Call
            </button>
          </div>
```
<p align=center><img src="https://user-images.githubusercontent.com/3941856/78079986-2308f400-7362-11ea-86bd-5c375e7e1805.png" alt="join call button" /></p>

We need a way to start the call. A button works for that. We hook up the `Join Call` button to `run()` from the `script.js` file.
```HTML
           <button
              onclick="run()"
            >
              Join Call
            </button>

```
2. The current participants list
<p align="center"><img src="https://user-images.githubusercontent.com/3941856/78086634-66b82980-7373-11ea-8940-3c7360d68965.png" alt="call participants"></p>

The local user is counted as a participant and is always shown. As more people join the call, their name and hand raised status will populate the list. When participants leave, they will be removed from the list on exit. This part of the page also contains the raise/lower hand button, so the local user can broadcast their hand state to the rest of the participants.

The `local-participant-info` `<div>` shows your call status (user name and if your hand is up). When callers join, their info will populate the `participant-list` `<ul>` as list items.
```HTML
          <div class="participants-section">
            <p class="title">Participants</p>
            <p class="subtitle">
              Raise your hand to get the moderator's attention
            </p>
            <div class="box local-participant-info ">
              <img
                id="hand-img"
                class=" is-pulled-right hidden"
                src="assets/hand.png"
                alt="hand icon"
              />
              <p class="name-label"><strong>You</strong></p>
              <button
                class="button is-primary hidden raise-hand-button"
                onclick="raiseOrLowerHand()"
              >
                raise your hand
              </button>
              <p class="hand-state has-text-right"></p>
            </div>
            <ul class="participant-list"></ul>
          </div>
```
3. The call frame

For this app, the call frame has a designated space in the user interface. It's set up in the code below. In the JavaScript portion, we will reference this iframe to wrap the Daily-js call frame.

![colorful illustrated people](https://user-images.githubusercontent.com/3941856/78086769-d201fb80-7373-11ea-9c7e-97769fa48738.png)
> The example app has a cute illustration behind the `call-frame` `<iframe>`. It gets covered up once a call starts.

```HTML
            <div class="call-panel tile is-child notification is-light">
              <iframe
                class="is-overlay"
                id="call-frame"
                title="daily.co call frame"
                allow="camera; microphone"
              ></iframe>
            </div>
```
> The [full sample app source code](https://repl.it/@prophen/NavyUniformRoute) is viewable on repl.it

## JavaScript time! Working with Daily-js
We're using the [daily-js front-end library](https://docs.daily.co/reference#using-the-dailyco-front-end-library) to capture and respond to call events and implement our custom raise a hand feature. 

Using the [sendAppMessage()](https://docs.daily.co/reference#%EF%B8%8F-sendappmessage) method allows our sample app to respond to events that happen during the call. 

Here are the steps we will follow in psuedocode:

> 
- Initialize the `localParticipant` and `handState` variables
- On Click of Join Call button call `run()`
- In `run()`
    - Initialize the `room` variable
    - Create the call frame (we use `.wrap()` in this example)
    - Listen for call events 
        - When the `localParticipant` joins a call, or another caller joins, updates, or leaves the call broadcast their current hand state with `callFrame.sendAppMessage()`
    - Join the call at the `room.url` within the sample app UI
- Once in a call, the `localParticipant` can raise and lower their hand regardless of in-call events and see it reflected in the sample app 
- When a message is received with other callers' handState, update the `handState` and reflect it in the participants list UI

### The `localParticipant` object
We initialize the `localParticipant` with `{ handRaised: false }`. That's the only property we need to initialize for our sample app because we will copy over the properties that daily-js provides in `joinedMeeting()`

```JavaScript
let localParticipant = {
  handRaised: false
};
```

### The Custom `handState` object
```JavaScript
let handState = {
  list: [],
  // sends the current hand state without raising or lowering the hand
  broadcastLocalHandState() {
    let data = {
      handRaised: localParticipant.handRaised,
      session_id: localParticipant.session_id
    };
    window.callFrame.sendAppMessage(data, "*");
  },
  addHandToList(e) {
    this.list = [...this.list].concat([
      { session_id: e.fromId, handRaised: e.data.handRaised }
    ]);
    updateParticipants();
  },
  removeHandFromList(e) {
    this.list.splice(this.list.indexOf(e.data.session_id), 1);
    updateParticipants();
  },
  // gets called when a message is received, adds the session id and the hand state to this list
  updateList(e) {
    e.data.handRaised
      ? handState.addHandToList(e)
      : handState.removeHandFromList(e);
  },
  // raise or lower the local hand
  toggleState() {
    localParticipant = {
      ...localParticipant,
      handRaised: !localParticipant.handRaised
    };
  }
};

### Bring in the call frame
When a caller clicks the Join Call button, `run()` is called and a Daily.co call is started and joined.
 
Here's where you need that Daily.co room URL. Initialize the room variable using this line:
```JavaScript
  let room = { url: "https://popschools.daily.co/qOrbXQ3zJZC7o7aH8ycI" };
```

If you have a free account, you can use the 'yoursubdomain.daily.co/hello' room that is created for you without modification.

Now we use the iframe we created in HTML. The .[wrap() method](https://docs.daily.co/reference#%EF%B8%8F-wrap) of the [DailyIframe](https://docs.daily.co/reference#the-dailyiframe-class) takes the DOM element of the iframe and an object for properties you'd like to alter. I wanted a leave button on the call interface, so I set `showLeaveButton: true`
```JavaScript
  window.callFrame = DailyIframe.wrap(document.getElementById("call-frame"), {
    showLeaveButton: true
  });
```
The final line of this function joins the room specified earlier. 

```JavaScript
  await callFrame.join({ url: room.url });
```
Here's the complete `run()`. We'll talk about the `on` events in the next section.
 
```JavaScript
async function run() {
  // setting up for conditional rendering
  const welcomePrompt = document.querySelector(".welcome-box");
  welcomePrompt.classList.toggle("hidden");

  let room = { url: "https://popschools.daily.co/qOrbXQ3zJZC7o7aH8ycI" };

  window.callFrame = DailyIframe.wrap(document.getElementById("call-frame"), {
    showLeaveButton: true
  });

  callFrame
    .on("joined-meeting", joinedMeeting)
    .on("left-meeting", leftMeeting)
    .on("participant-joined", participantJoined)
    .on("participant-left", updateParticipants)
    .on("app-message", messageReceived);

  // join the room
  await callFrame.join({ url: room.url });
}
```
### Listening for call events
The only way to respond to in-call events is to use [event](https://docs.daily.co/reference#events) callbacks. We set these up in `run()` to listen for messages (the way we raise or lower a hand) and to respond when other people leave or join (so that everyone sees everyone else's hand state)

```JavaScript
callFrame
    .on("joined-meeting", joinedMeeting)
    .on("left-meeting", leftMeeting)
    .on("participant-joined", participantJoined)
    .on("participant-left", updateParticipants)
    .on("app-message", messageReceived);
```
### The local hand state - raising a hand
![local hand state](https://user-images.githubusercontent.com/3941856/78180782-21016c80-7418-11ea-9271-4e02278905f6.png)

We have to write custom code to show hands raised or lowered during the call. Also, when a new person joins, we need them to be able to see who in the call had their hands raised before they got there. 

Remember, we communicate between the sample app and the callFrame using `callFrame.sendAppMessage()` and the `.on` event callbacks.

Every message sent sends the `localParticipant.handRaised` property value. The messages go out to all other participants and that's how they know if the local participant has raised or lowered their hand.

The `setTimeout()` is added to make sure that the message doesn't send before the new caller has joined and can receive it. Messages are only delivered to participants in-call.

```JavaScript
async function raiseOrLowerHand() {
  handState.toggleState();
  
  // Let other users see my hand state by sending a message
  handState.broadcastLocalHandState();
  updateParticipants();

  // Update the UI to show my own hand state
  document.getElementById("hand-img").classList.toggle("hidden");
  document.querySelector(".raise-hand-button").innerText = `${
    localParticipant.handRaised ? "lower your hand" : "raise your hand"
    }`;
  document.querySelector(".hand-state").innerText = `${
    localParticipant.handRaised ? "your hand is up" : "your hand is down"
    }`;
}

// Send a message to update everyone's hand state
async function messageReceived(e) {
  handState.updateList(e);
}

async function joinedMeeting(e) {
  localParticipant = {
    ...e.participants.local,
    handRaised: false
  };

  let localParticipantInfoBox = document.querySelector(
    ".local-participant-info"
  );
  localParticipantInfoBox.innerHTML = `
    <img
      ...
    />  

    <p class="name-label"><strong>${localParticipant.user_name ||
    "You"}</strong></p>
    <button
      class="button is-info hidden raise-hand-button"
      onclick="raiseOrLowerHand()"
      >raise your hand
    </button>
    <p class="hand-state has-text-right">your hand is down</p>
`;
  await setTimeout(handState.broadcastLocalHandState, 2000);
  await setTimeout(() => {
    document.querySelector(".raise-hand-button").classList.toggle("hidden")
  }, 2000)
}
```

### See everyone else's hands up
![call participant list](https://user-images.githubusercontent.com/3941856/78181291-f532b680-7418-11ea-97c6-25d1961f4d29.png)

The way our sample app knows if another caller's hand goes up is by receiving a message. When that message is received with the call participant's hand state we update the UI in `updateParticipants()`

`updateParticipants()` gets the current list of particpants from `callframe.participants()`, goes through each caller, checks if their hand is raised, and updates the UI accordingly.

```JavaScript
function updateParticipants(e) {
  // the local user has their own hand state. Other callers raised hands are saved in the handState list
  let raisedHands = handState.list.map(caller => caller.session_id);
  let ps = callFrame.participants();
  // will append the li elements (call participants) to this list
  let list = document.querySelector(".participant-list");
  // clear so the list shows current info when participants enter or leave
  list.innerHTML = "";

  Object.keys(ps).forEach(p => {
    let participant = ps[p];
    let callerHandUp = raisedHands.includes(participant.session_id);
    // This li will display the single participant info
    let li = document.createElement("li");
    // Don't list the local user again. Their info is in the first box.
    if (participant.local) {
      return;
    }
    let handStateLabel = callerHandUp ? "hand up" : "";
    li.innerHTML =
      `<div class="box">
        ${raisedHands.includes(participant.session_id)
        ? `<img
          id="hand-img"
          class=" is-pulled-right"
          src="assets/hand.png"
          alt="hand icon"
          />`
        : ""
      }
        <p>${participant.user_name || "Guest"}</p>
        <p class="hand-state has-text-right">${handStateLabel}</p> 
      </div>`;
    list.append(li);
  });
}
```
> The [full sample app source code](https://repl.it/@prophen/NavyUniformRoute) is viewable on repl.it

## Thanks for reading this
I hope you found it useful.
