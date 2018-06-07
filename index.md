### Tools

The examples are illustrated using the following command-line tools:

* [httpie](https://httpie.org/)
* [pbpaste](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/pbpaste.1.html)
* [jq](https://stedolan.github.io/jq/)

- - -

### Getting and modifying the site configuration

First, get your `<siteKey>` from the Houm app main view.

To get the site configuration:

```bash
http get https://houmkolmonen.herokuapp.com/api/site/<siteKey>
```

The response is the site configuration (a.k.a. `site.json`).

To modify the configuration, copy the modified `site.json` to clipboard and

```bash
pbpaste | http put https://houmkolmonen.herokuapp.com/api/site/<siteKey>
```

### Controlling individual devices

To list the devices in `site.json`, do

```bash
http get https://houmkolmonen.herokuapp.com/api/site/<siteKey> | jq '.devices[] | { id: .id, name: .name }'
```

Pick the device you want to control. Then, write a state for the device, such as

```json
{
  "id": "<deviceId>",
  "state": {
    "on": true,
    "bri": 255
  }
}
```

and copy it into clipboard. Then

```bash
pbpaste | http post https://houmkolmonen.herokuapp.com/api/site/<siteKey>/applyDevice
```

### Controlling all devices in a room

Find the room id of the room from `site.json`, then write a state for the devices in the room, such as

```json
{
  "roomId": "<roomId>",
  "state": {
    "on": true,
    "bri": 255
  }
}
```

and copy it into clipboard. Then

```bash
pbpaste | http post https://houmkolmonen.herokuapp.com/api/site/<siteKey>/applyRoom
```

### Activating scenes

To list the scenes in `site.json`, do

```bash
http get https://houmkolmonen.herokuapp.com/api/site/<siteKey> | jq '.scenes[] | { id: .id, name: .name }'
```

Pick the scene you want to activate, then

```bash
echo '{ "id": "<sceneId>" }' | http post https://houmkolmonen.herokuapp.com/api/site/<siteKey>/applyScene
```

To activate the "All on" or "All off" scene, do

```bash
echo '{ "id": "allOn" }' | http post https://houmkolmonen.herokuapp.com/api/site/<siteKey>/applyScene
```

or

```bash
echo '{ "id": "allOff" }' | http post https://houmkolmonen.herokuapp.com/api/site/<siteKey>/applyScene
```

### Websocket API

The example below requires `node` 6.x or above.

```javascript
const io = require('socket.io-client')

const SITEKEY = 'we-take-the-ball-to-the-build'

const socket = io.connect('https://houmkolmonen.herokuapp.com', {
  reconnectionDelay: 1000,
  reconnectionDelayMax: 3000,
  transports: ['websocket'],
})

// First, subscribe to you site.

socket.emit('subscribe', { siteKey: SITEKEY })

// In response to the subscribe message, you'll get a 'siteKeyFound' or 'noSuchSiteKey' message.

socket.on('siteKeyFound', ({ siteKey, data }) => {
  // ...
})

socket.on('noSuchSiteKey', ({ siteKey }) => {
  // ...
})

// You'll receive a 'siteKeyExpired' message if the key you are using is deleted or it expires.

socket.on('siteKeyExpired', ({ siteKey }) => {
  // ...
})

// If the central unit is disconnected from the cloud server, you'll receive an 'offline' message.

socket.on('offline', ({ siteKey }) => {
  // ...
})

// When the site configration changes, you'll receive a 'site' message.

socket.on('site', ({ siteKey, data }) => {
  // Do something with the new site configuration
})

// When a switch is pressed, you'll receive a 'peripheralInput' message.

socket.on('peripheralInput', ({ siteKey, data }) => {
  // Do something with the peripheral input data
})

// When sensor data is available, you'll receive a 'sensorData' message.

socket.on('sensorData', ({ siteKey, data }) => {
  // Do something with the sensor data
})

// The following messages should be emitted only after receiving a 'siteKeyFound' message.

socket.emit('apply/device', {
  siteKey: SITEKEY,
  data: {
    id: '<deviceId>',
    state: { on: true, bri: 255 },
  },
})

socket.emit('apply/room', {
  siteKey: SITEKEY,
  data: {
    roomId: '<roomId>',
    state: { on: true, bri: 255 },
  },
})

socket.emit('apply/scene', {
  siteKey: SITEKEY,
  data: {
    id: '<sceneId>',
  },
})
```
