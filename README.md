# @runnane/node-red-contrib-easee npm module

[![npm](https://img.shields.io/npm/v/@runnane/node-red-contrib-easee.svg?maxAge=2592000)](https://www.npmjs.com/package/@runnane/node-red-contrib-easee)
[![downloads](https://img.shields.io/npm/dt/@runnane/node-red-contrib-easee.svg?maxAge=2592000)](https://www.npmjs.com/package/@runnane/node-red-contrib-easee)

Node-Red module for streaming Easee charger data.

## Features

- SignalR streaming client
- Pre-defined list of REST API GET/POST commands
- Custom commands through REST API

## Howto

`npm i @runnane/node-red-contrib-easee`

Add the `easee Charger Streaming Client` node
Configure the node with username/password and the Charger ID.

## Streaming node

Configure the node with username/password and a Charger ID ("EH000000").
Streaming telemetry from the signalR enpoint will be available in the fourth output,
the `ProductUpdate` one.

## REST node

Use the `easee REST Client` node
Configure the node with an account username/password.
The REST node will not authenticate on its own, so you will need to authenticate/renew tokens.
However, if you use the `easee Charger Streaming Client` node,
you do not need to authenticate additionally with the REST node, as the signalR socket
will authenticate and renew automatically.

There are two ways of sending commands:

### Sending predefined commands via topic

Send your command via topic into the node.
You can set the charger, site and/or circuit variables directly in the node, or send them as
`msg.charger`, `msg.site` and `msg.circuit` to override.
Implemented commands that may be sent via topic include:

- `login`
- `refresh_token`
- `charger`
- `charger_details`
- `charger_state`
- `charger_site`
- `charger_config`
- `charger_session_latest`
- `charger_session_ongoing`
- `stop_charging`
- `start_charging`
- `pause_charging`
- `resume_charging`
- `toggle_charging`
- `dynamic_current` (Without msg.payload.body for reading (GET), and with msg.payload.body for setting (POST).)
- `reboot`

Example, [get charger details](https://developer.easee.com/reference/get_api-chargers-id-details):

```javascript
node.send({
  topic: "charger_details",
  charger: "EH000000",
});
```

### Sending custom commands

To send custom commands to the Easee API, you'll need to use a function node in Node-RED that feeds into the "easee REST Client" node. This method allows for more flexibility in interacting with the API, enabling you to access endpoints that aren't covered by the predefined commands.

To construct a custom command, create a function node with the following structure:

```javascript
msg.payload = {
    method: 'GET' or 'POST',
    path: '/api-endpoint-path',
    body: {} // Include for POST requests, omit for GET
};
return msg;
```

The `method` field specifies whether it's a GET or POST request. The `path` field should contain the API endpoint you want to access. For POST requests, include a `body` object with the necessary parameters.

To find available API endpoints, refer to the Easee API documentation at https://developer.easee.com/reference. This resource provides a comprehensive list of endpoints and their required parameters.

#### Example 1: Setting a weekly charge plan:

```javascript
msg.payload = {
    method: 'POST',
    path: '/chargers/EH00000/weekly_charge_plan',
    body: {
        "isEnabled": true,
        "days": [
            {
                "dayOfWeek": 0,
                "ranges": [
                    {
                        "chargingCurrentLimit": 32,
                        "startTime": "10:00",
                        "stopTime": "11:00"
                    }
                ]
            }
        ]
    }
};
return msg;
```

#### Example 2: Adjusting the dynamic charger current to 8 Ampere:

```javascript
msg.payload = {
    method: 'POST',
    path: '/chargers/EH00000/settings',
    body: { "dynamicChargerCurrent": 8 }
};
return msg;
```

## Example

See [example flows](https://github.com/runnane/node-red-contrib-easee/blob/main/example.json)
![image](https://github.com/runnane/node-red-contrib-easee/assets/1679504/744fd250-3bab-46d8-a31a-3421f6d4c42d)

## Credits and references

- Initially forked from [node-red-contrib-signalrcore](https://github.com/scottpage/node-red-contrib-signalrcore), then rewritten
- REST API documentation [developer.easee.com](https://developer.easee.com/docs/integrations)
- Enumerations [developer.easee.com](https://developer.easee.com/docs/enumerations)
