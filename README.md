### Demo Application

In this tutorial, we’ll show how to build simple audio/video conferencing application with Drum.

You can see what we’ll be building here: [Final Result](https://codepen.io/p8952/pen/bOMOBZ). If the code doesn’t make sense to you, don’t worry! The goal of this tutorial is to help you understand Drum.

We’ll assume that you have some familiarity with HTML and JavaScript, but you should be able to follow along even if you’re coming from a different programming language.

First, open this [Starter Code](https://codepen.io/p8952/pen/WLyqQO) in a new tab. The new tab should display an empty JS code editor, we will be writing our code here in this tutorial.

You might want to follow along with a friend, or in two seperate tabs so you can see how everything looks & works with mutiple participants!

### DOM Manipulation

To get started we will define some functions to handle DOM manipulation. For simplicity these are written in vanilla JavaScript and interact with the DOM directly. In a real application they will most probably be tightly coupled to your user interface code and be written using a library such as React, Angular, or Vue.

```
function getMediaStream() {
  return navigator.mediaDevices.getUserMedia({ audio: true, video: true });
}

function addVideoElement(id, stream) {
  if (!document.getElementById(id)) {
    let videoElement = document.createElement("video");
    videoElement.classList.add("drum-video");
    videoElement.id = id;
    videoElement.autoplay = true;
    videoElement.srcObject = stream;
    videoElement.width = 300;
    document.body.appendChild(videoElement);
  }
}

function getVideoElements() {
  return Array.from(document.getElementsByClassName("drum-video"));
}

function removeVideoElement(id) {
  if (document.getElementById(id)) {
    let videoElement = document.getElementById(id);
    videoElement.parentNode.removeChild(videoElement);
  }
}
```

`getMediaStream()`: Returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) that resolves with a [MediaStream](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream) containing an audio track (user's microphone) and video track (user's webcam)

`addVideoElement()`: Creates a [Video Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) and adds it to the DOM

`getVideoElements()`: Returns an [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) containing all of the [Video Elements](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) in the DOM that have been created by `addVideoElement()`

`removeVideoElement()`: Removes a [Video Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) from the DOM

### Defining Constants

Once we have the DOM manipulation sorted we can define some constants that will be used to connect to Drum. In a real application these will generally be unique per user and most probably be generated based on pre-existing information you have about the user. 

```
const applicationId = "9999999999";
const roomId = parseInt(`${applicationId}55555`, 10);
const userId = `${Math.floor(Math.random() * 899999) + 100000}`;
const userName = "Summer Alaska";
```

`applicationId`: For the purpose of this demo this value is hard-coded to `9999999999`, in a real application it will be a 10 digit [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) you are given along with your API Key

`roomId`: Should be [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) prefixed with your applicationId, any users connecting with the same value will be placed in the same room

`userId`: Should be a [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) that can uniquely identify the user

`userName:` Should be a [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) that can visually identify the user

### Connecting to Drum

As a next step we want to initialise Drum API and create a connection.

```
DrumMediaPlatform.init()
  .then(() => {
    return DrumMediaPlatform.connect(roomId, userId, userName);
  })
  .catch(error => {
    console.error("DRUM -", error);
  });
```

We also want add an [EventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventListener) that will call an [EventHandler](https://developer.mozilla.org/en-US/docs/Web/API/EventListener/handleEvent) whenever the connections in a room change. It is here that we'll define most of our business logic and decide what we can to do with the connections in a room. 

```
DrumMediaPlatform.on(DrumMediaPlatform.events.ROOM_UPDATE, event => {
});
```

Before continuing we need to familiarise ourselves with a couple of Drum concepts.

### Media Architecture

Drum uses a hybrid media architecture.

Audio mixing is handled by an [MCU](https://webrtcglossary.com/mcu/) and any user broadcasting their audio will automatically receive back the audio from all other users.

Video mixing is handled by an [SFU](https://webrtcglossary.com/sfu/) and a user broadcasting their video will not automatically receive back the video from other users. You need to subscribe to the video connections of one more more users to receive video from them.

### Audio Conferencing

Now that we've covered those concepts we can return to our code.

The first thing we want to do is use the `getMediaStream()` function to retrieve a [MediaStream](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream) containing two tracks, one audio (the user's microphone) and one video (the user's webcam)

Drum provides two functions to handle broadcasting the tracks contained within a [MediaStream](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream)

* `addAudioStream()`: Extracts the audio track from a [MediaStream](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream) and broadcasts it

* `addVideoStream()`: Extracts the video track from a [MediaStream](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream) and broadcasts it

For now we'll just focus on the audio track.

```
DrumMediaPlatform.init()
  .then(() => {
    return DrumMediaPlatform.connect(roomId, userId, userName);
  })
+  .then(() => {
+    return getMediaStream();
+  })
+  .then(mediaStream => {
+    return DrumMediaPlatform.addAudioStream(mediaStream, "microphone");
+  })
  .catch(error => {
    console.error("DRUM -", error);
  });
```

With this in place we will be broadcasting the audio from our microphone and receiving back the audio from all other user's microphones. If we are the only user broadcasting audio then we will instead receive back music on hold.

🎉 Congratulations! You've built an audio conferencing application!

### Video Conferencing

Video conferencing requires a little bit more work, but not much!

To start with we'll also broadcast the video track from our [MediaStream](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream).

```
DrumMediaPlatform.init()
  .then(() => {
    return DrumMediaPlatform.connect(roomId, userId, userName);
  })
  .then(() => {
    return getMediaStream();
  })
  .then(mediaStream => {
-    return DrumMediaPlatform.addAudioStream(mediaStream, "microphone");
+    return Promise.all([
+      DrumMediaPlatform.addAudioStream(mediaStream, "microphone"),
+      DrumMediaPlatform.addVideoStream(mediaStream, "camera")
+    ]);
  })
  .catch(error => {
    console.error("DRUM -", error);
  });
```

We'll now need to select which video connections we want to subscribe to. Remember the [EventHandler](https://developer.mozilla.org/en-US/docs/Web/API/EventListener/handleEvent) we created earlier? This will fire whenever the state in the room changes - for example when a new video connection becomes available to subscribe to.

Drum provides a function to handle subscribing to a video connection.

* `subscribe()`: Subscribes to a video connection

```
DrumMediaPlatform.on(DrumMediaPlatform.events.ROOM_UPDATE, event => {
+  const connections = event.connections;
+
+  Object.keys(connections).forEach(connectionId => {
+    const connection = connections[connectionId];
+
+    if (connection.type === "video" && !connection.subscribed) {
+      connection.subscribe().catch(error => {
+        console.error("DRUM -", error);
+      });
+    }
+  });
});
```

Great! We will now subscribe to every new video connection that appears in the room! However simply subscribing to a video connection will not make it visible, it's down to your application to decide how and where it wants to render this video.

We can use the DOM manipulation functions we defined earlier to take all the videos we are subscribed to and render them to the DOM.

```
DrumMediaPlatform.on(DrumMediaPlatform.events.ROOM_UPDATE, event => {
  const connections = event.connections;

  Object.keys(connections).forEach(connectionId => {
    const connection = connections[connectionId];

    if (connection.type === "video" && !connection.subscribed) {
      connection.subscribe().catch(error => {
        console.error("DRUM -", error);
      });
    }

+    if (connection.type === "video" && connection.stream) {
+      addVideoElement(connection.id, connection.stream);
+    }
  });
});
```

Amazing! Now every time someone joins the room their video connection will be subscribed to, and when that subscription is successful and the video stream becomes available their video will be rendered in the DOM.

We're almost there, but there is one last thing to do. Unfortunately sometimes people leave video conferences and our meetings have to end 😢. As things currently stand, when people leave we'll be left with empty videos rendered in the DOM.

Luckily our [EventHandler](https://developer.mozilla.org/en-US/docs/Web/API/EventListener/handleEvent) is called whenever anything changes in the room, including people leaving. So whenever the state in the room changes we can use that as an opportunity to remove any empty videos from the DOM.

```
DrumMediaPlatform.on(DrumMediaPlatform.events.ROOM_UPDATE, event => {
  const connections = event.connections;

  Object.keys(connections).forEach(connectionId => {
    const connection = connections[connectionId];

    if (connection.type === "video" && !connection.subscribed) {
      connection.subscribe().catch(error => {
        console.error("DRUM -", error);
      });
    }

    if (connection.type === "video" && connection.stream) {
      addVideoElement(connection.id, connection.stream);
    }
  });


+  getVideoElements().forEach(videoElement => {
+    if (!connections[videoElement.id] || !connections[videoElement.id].stream) {
+      removeVideoElement(videoElement.id);
+    }
+  });
});
```

[That's all folks!](https://www.youtube.com/watch?v=b9434BoGkNQ) There is a lot more we might want to add to this application, and there is a LOT more Drum can do. But we hope this has given you a taste and that you now feel like you have a decent grasp on how Drum works.
