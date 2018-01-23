# Sensit workshop

How to add a Sensit on Sigfox Platform

## About this workshop

The goal of this workshop is to understand how to get started with Sigfox.
For this workshop, we will be using a Sensit (Sigfox device with various sensors).
More info on

The different steps will be:

* Send your first Sigfox message
* See your message in the Sigfox backend
* Deploy an open source platform.
* Set a callback to push your data to this platform.
* Parse a Sigfox payload.
* Understand Sigfox Geolocation service.

## Send you first Sigfox message

You've been provided a Sensit.
![Sensit](screenshots/sensit-2.jpeg)
This is a Sigfox device that include multiple sensors:
- Temperature sensor
- Humidity sensor
- Light sensor
- Accelerometer
- Magnet (Hall effect)
- Button

In this workshop, we won't see how to reprogram this device using its SDK but feel free to have a look at [build.sigfox.com](https://build.sigfox.com/sensit-for-developers).

Let's start with sending a message: Just double press the button.
That's it!

## See your messages in Sigfox Backend

Now go to [Sigfox Backend](https://backend.sigfox.com) and login with the credentials you've been given.
Click on the device menu and select your device.

![device info](screenshots/backend-device-info.png)

Now click on message, you should see your message:

![Sigfox Backend](screenshots/backend-device-messages.png)

Cool right?
Okay, but what can I do with this message?

The next step will be to deploy an open source platform to see and use this Sigfox messages.

## Sigfox Platform


### Option 1: Test with the demo application

[Demo](https://sigfox-platform.thenorthweb.com)

### Option 2: Deploy your own instance with [Heroku](https://heroku.com)

Deploy an instance on your Heroku account to play around with it!

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/luisomoreau/sigfox-platform/tree/v0.1-alpha)

An alternative way to get it running at Heroku is to install the [Heroku Toolbelt](https://heroku.com/deploy?template=https://github.com/luisomoreau/sigfox-platform/) and follow these steps:

```
git clone https://github.com/luisomoreau/sigfox-platform.git my-project
cd my-project
heroku apps:create my-project
git push heroku master
```

If you are not familiar with Heroku, just create an account an follow the procedure:

1. **Create a new app:**

![create app](screenshots/deploy-1.png)

2. **Build & deploy app:**

![build app](screenshots/deploy-2.png)

3. **(Optional) Link the application with a MongoDB MLab database (Free):**

*Note that if you don't link a database to your application, all the data will be erased every time the application restarts.*

* Go to [https://mlab.com](https://mlab.com/login/) and create an account and login.

* Create a new MongoDB Deployments:

![mlab-select-service](screenshots/mlab-select-service.png)

* Select your plan:

![mlab-select-plan](screenshots/mlab-select-plan.png)

* Select your region:

![mlab-select-region](screenshots/mlab-select-region.png)

* Create database:

![mlab-create-db](screenshots/mlab-create-db.png)

* Validate:

![mlab-validate](screenshots/mlab-validate.png)

* Create database user:

![mlab-create-user](screenshots/mlab-create-user.png)

* Copy your MongoDB URI :

![mlab-view-user](screenshots/mlab-view-user.png)

* Go back to your Heroku Dashboard and go to the Settings tab:

![heroku-show-env-variables](screenshots/heroku-show-env-variables.png)

* Click on Reveal Config Vars and add your MongoDB URI:

![heroku-add-MONGODB_URI](screenshots/heroku-add-MONGODB_URI.png)

* Restart all dynos:

![heroku-restart-dynos](screenshots/heroku-restart-dynos.png)


### User guide

- Open app and register:

![login](screenshots/login.png)

![register](screenshots/register.png)

![login-2](screenshots/login-2.png)

Note that, the first user to register will be granted an admin role.
The other users to register will be granted user roles.

When you logged in successfully, you will arrive on the overview page. At this stage, it should be empty:

![overview](screenshots/overview-empty.png)

Now we want to create a callback from the Sigfox Backend to push incoming messages to the platform:

Navigate to the Connectors tab and create a developer access token:

![create-access-token](screenshots/create-dev-access-token.png)

Now go back to the Sigfox Backend and click on the INFORMATION tab of your device:

![device-info](screenshots/backend-device-info.png)

Click on the CALLBACKS tab:

![create-callback](screenshots/backend-view-callbacks.png)

Create a new callback:

![new-callback](screenshots/new-callback.png)

Copy past the information provided in the platform:

![uplink-callback](screenshots/connector-create-uplink-callback.png)

![backend-uplink-callback](screenshots/uplink-callback.png)

Send a Sigfox message again with your device and view it on the platform:

![overview-first-message](screenshots/overview-first-message.png)

Add another callback to use the Sigfox geolocation service:

![geoloc-callback](screenshots/connector-create-geoloc-callback.png)

![backend-geoloc-callback](screenshots/geoloc-callback.png)

The Geolocation information may take up to 10 seconds to arrive. This is the necessary time for all the messages to be received by Sigfox Backend and to process the Geolocation service.

## Decode Sigfox Payload

Go back on the platform and add a parser to decode the Sigfox Payload:

![add-parser](screenshots/add-parser.png)

**Let's focus on how to add your new parser in order to decode custom payloads.**

You must respect a particular nomenclature such as the payload variable name, the geolocation structure _(if any)_ and the result of the function you will write.

### 1. The payload variable name: `var payload`

This variable will automatically contain the Sigfox message hexadecimal frame used for your parsing, make sure to name it as above.

### 2. The result of the parser

This has to be an **array of objects**. Each object has to follow this structure:

* key
* value
* type
* unit

```javascript
var obj = {};
obj.key = ; //(string) a key
obj.value = ; //(any) a value
obj.type = ; //(string) a type (string, number, boolean)
obj.unit = ; //(string) a unit or none ('°C', '%', '')
```

If the payload cointains geolocation you have to store a special object with the 'geoloc' key. Below is an example for GPS coordinates:

```javascript
obj.key = 'geoloc';
obj.value = 'GPS';
obj.type = 'string';
obj.unit = '';
parsedData.push(obj);
obj = {};
obj.key = 'lat';
obj.value = 48.858093;
obj.type = 'number';
obj.unit = '';
parsedData.push(obj);
obj = {};
obj.key = 'lng';
obj.value = 2.294694;
obj.type = 'number';
obj.unit = '';
parsedData.push(obj);
```

### Example
```javascript
var payload,
  temperature,
  parsedData = [],
  obj = {};

// If byte #1 of the payload is temperature (hex to decimal)
temperature = parseInt(payload.slice(0, 2), 16);

// Store objects in parsedData array
obj = {};
obj.key = 'temperature';
obj.value = temperature;
obj.type = 'number';
obj.unit = '°C';
parsedData.push(obj);

//console.log(parsedData);
return parsedData;
```

Returns:

If the hexadecimal payload is '11'.
```javascript
[
  {
    "key": "temperature",
    "value": 17,
    "type": "number",
    "unit": "°C"
  }
]
```

Now let's try to decode the Sensit payload.
Here you will find the payload description:
[Sensit payload description](screenshots/Sensit-Payload-Decoding-Guide.pdt)

To simplify the task, you will only need to decode the light, the temperature and humidity.
Here is the base parser:

Copy paste the following code into the new parser:
```
var payload,
    battery,
    type,
    timeFrame,
    mode,
    humidity,
    temperature,
    light,
    alert,
    firmwareVersion,
    parsedData = [],
    obj = {};

// Byte #1
var byte = parseInt(payload.slice(0, 2), 16).toString(2);
while (byte.length < 8)
    byte = '0' + byte;
battery = byte.slice(0, 1);
type = parseInt(byte.slice(1, 3), 2);
switch (type) {
    case 0:
        type = 'Classic';
        break;
    case 1:
        type = 'Button';
        break;
    case 2:
        type = 'Alert';
        break;
    case 3:
        type = 'New Mode';
        break;
    default:
        type = 'Unknown {' + type + '}';
}
timeFrame = parseInt(byte.slice(3, 5), 2);
switch (timeFrame) {
    case 0:
        timeFrame = '10 minutes';
        break;
    case 1:
        timeFrame = '1 hour';
        break;
    case 2:
        timeFrame = '6 days';
        break;
    case 3:
        timeFrame = '24 hours';
        break;
    default:
        timeFrame = 'Unknown {' + timeFrame + '}';
}
mode = parseInt(byte.slice(5, 8), 2);
switch (mode) {
    case 0:
        mode = 'Button';
        break;
    case 1:
        mode = 'Temperature & Humidity';
        break;
    case 2:
        mode = 'Light';
        break;
    case 3:
        mode = 'Door';
        break;
    case 4:
        mode = 'Move';
        break;
    case 5:
        mode = 'Reed switch';
        break;
    default:
        mode = 'Unknown mode {' + mode + '}';
}

// Byte #2
var byte = parseInt(payload.slice(2, 4), 16).toString(2);
while (byte.length < 8)
    byte = '0' + byte;
battery += byte.slice(4, 8);
battery = (parseInt(battery, 2) * 0.05 + 2.7).toFixed(2);

// Temperature & Humidity
if (mode === 'Temperature & Humidity') {

  //Try to decode the temperature and the humidity here!
  temperature = 0;
  humidity = 0;

}

// Light
if (mode === 'Light') {
    //Try to decode the light here!
    light = 0;
}

// Alert (Door - Move - Reed switch)
if (mode === 'Door' || mode === 'Move' || mode === 'Reed switch')
// Byte #4
    alert = parseInt(payload.slice(6, 8), 16) === 0 ? false : true;


// Store objects in parsedData array
obj = {};
obj.key = 'type';
obj.value = type;
obj.type = 'string';
obj.unit = '';
parsedData.push(obj);
obj = {};
obj.key = 'timeFrame';
obj.value = timeFrame;
obj.type = 'string';
obj.unit = '';
parsedData.push(obj);
obj = {};
obj.key = 'mode';
obj.value = mode;
obj.type = 'string';
obj.unit = '';
parsedData.push(obj);
obj = {};
obj.key = 'firmwareVersion';
obj.value = firmwareVersion;
obj.type = 'string';
obj.unit = '';
parsedData.push(obj);
obj = {};
obj.key = 'temperature';
obj.value = temperature;
obj.type = 'number';
obj.unit = '°C';
parsedData.push(obj);
obj = {};
obj.key = 'humidity';
obj.value = humidity;
obj.type = 'number';
obj.unit = '%';
parsedData.push(obj);
obj = {};
obj.key = 'light';
obj.value = light;
obj.type = 'number';
obj.unit = 'lux';
parsedData.push(obj);
obj = {};
obj.key = 'alert';
obj.value = alert;
obj.type = 'boolean';
obj.unit = '';
parsedData.push(obj);
obj = {};
obj.key = 'battery';
obj.value = battery;
obj.type = 'number';
obj.unit = 'V';
parsedData.push(obj);

//console.log(parsedData);
return parsedData;
```

Now go to Device and click on edit:

![devices-edit](screenshots/devices-edit.png)

You now can see the decoded payload in the message view or in the overview:

![overview-last-message](screenshots/overview-last-message.png)

![messages](screenshots/messages.png)

And see the graphs:

![graph](screenshots/graph.png)


## Additional content

* [Framboise314](http://www.framboise314.fr/carte-de-prototypage-sigfox-par-snoc/)

* [Tutos Instructables](www.instructables.com/member/luisomoreau/)

* [Tutos Hackster](https://www.hackster.io/luisomoreau)
