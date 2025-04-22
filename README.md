# Nielsen DOM-less SDK

## License

Nielsen SDK contains material that is protected by copyright laws, patent laws, trade secret laws, and by international treaty provisions and is Copyright © 2024 The Nielsen Company (US) LLC. All intellectual property rights and licenses therein are reserved by The Nielsen Company (US) LLC and its licensors. Please read the license agreement presented [here](https://engineeringportal.nielsen.com/docs/Special:ClickThrough), which must be accepted in order to download the Nielsen SDKs.
For more information, reach out to your Nielsen Technical Account Manager(TAM).

## Overview

This project is an API that allows our clients to integrate the Nielsen SDK in DOM-less environments, e.g., React Native, Node, etc.

## Installation

Install with `npm install https://github.com/NielsenDigitalSDK/bsdk-domless#1.1.0` and import the `BsdkInstance` into video player component

`import { BsdkInstance } from 'bsdk-domless'`

## Exposed Interface

The exposed interface is as follows:

1. `ggPM` - method to send messages to the Nielsen SDK
2. `processEvent` - method to send app state to the Nielsen SDK, e.g., *focus, blur, appclose*

### Events that can be passed to `ggPM` method

1. `loadmetadata` - This event is used to send DCR Video metadata to the Nielsen SDK. This event should be called when the video metadata is loaded.

```javascript
instance.ggPM('loadmetadata', {
  type: 'content',
  assetid: 'VID-123456',
  program: 'program name',
  title: 'episode title with season and episode number',
  length: 'length in seconds',
  airdate: '20210321 09:00:00',
  isfullepisode: 'y',
  adloadtype: '2',
  segB: 'custom segment B', // optional
  segC: 'custom segment C', // optional
  crossId1: 'Standard Episode ID', // optional
  crossId2: 'Content Originator', //optional
});
```

2. `setplayheadposition` - This event is used to send the current playhead position to the Nielsen SDK. This event should be called when the video is playing.

```javascript
instance.ggPM('setplayheadposition', 10);
```

3. `end` - This event is used to send the end event to the Nielsen SDK. This event should be called when the video playback is finished, passing the playhead at that time.

```javascript
instance.ggPM('end', 300);
```

4. `play` - This event is used to send the play event to the Nielsen SDK. This event should be called when the video is played; often used when resuming from pause, passing the playhead at that time.

```javascript
instance.ggPM('play', 30);
```

5. `stop` - This event is used to send the stop event to the Nielsen SDK. This event should be called when transitioning from ads to content and content to ads, passing the playhead at that time.

```javascript
instance.ggPM('stop', 120);
```

6. `sendid3` - This event is used to send the id3 event to the Nielsen SDK. This event should be called when the id3 event is triggered, passing the id3 data from the stream.

```javascript
instance.ggPM('sendid3', '<id3 metadata received>');
```

### Events that can be passed to and processed by `processEvent` method

1. `blur` - This event should be passed to processEvent when the app goes to the background.

```javascript
instance.processEvent({ type: 'blur', timestamp: Date.now() });
```

2. `focus` - This event should be passed to processEvent when the app goes to the foreground.

```javascript
instance.processEvent({ type: 'focus', timestamp: Date.now() });
```

3. `appclose` - This event should be passed prior to closing the app.

```javascript
instance.processEvent({ type: 'appclose', timestamp: Date.now() });
```

## SDK Initialization

### status.ok()

Initialization and check for SDK instance can be done with `status.ok()` function

```javascript
const instance = await new BsdkInstance(
  appID,
  instanceName,
  instanceMetadata,
  implementationHooks
);

if (instance && instance.status.ok()) {
  expect(instance).not.toBe(undefined);
  expect(instance.status.ok()).toBe(true);
}
```

## Sample Nielsen BSDK-Domless NodeJS Example

- [Nielsen bsdk-domless NodeJS Repository Example](https://github.com/NielsenDigitalSDK/bsdk-domless-samples/tree/main/nodejs)

## Sample DCR Video Integration

```javascript
import { BsdkInstance } from 'bsdk-domless';

const nsdkConfig = {
    app_id: 'PXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX',
    instance_name: 'videoInstance',
};

const implementationHooks = {
    Log: {
		info: function (log: string) {
		  console.info(log);
		},
		debug: function (log: string) {
		  console.debug(log);
		},
		warn: function (log: string) {
		  console.warn(log);
		},
		error: function (error: string) {
		  console.error(error);
		}
	  },
	Storage: {
		setItem: async function (key: any, value: string) {
            /**
             * Sets a string value for given key. This operation can either modify an existing entry, if it did exist for given key, or add new one otherwise.
             * In order to store object value, you need to serialize it, e.g. using JSON.stringify().
            */
		},
		getItem: async function (key: any) {
            /**
             * Gets a string value for given key. This function can either return a string value for existing key or return null otherwise.
             * In order to store object value, you need to deserialize it, e.g. using JSON.parse().
            */
		},
		removeItem: async function (key: any) {
            /**
             * Removes item for a key, invokes (optional) callback once completed.
            */
		}
	  },
        Fetch: async function (url: string | URL | Request, options: any) {
            /**
             * We require that client pass in User-Agent header via options in Fetch request
            */
            const clientOpts = {
                headers: {
                    "User-Agent": "react-native-domless/1.6.7.42 Dalvik/2.1.0 (Linux; U; Android 5.1.1; Android SDK built for x86 Build/LMY48X)"
                }
            }
            const data = Object.assign(options, clientOpts);
            const response = await fetch(url, data);
            if (response.ok) {
                return response;
            } else {
                throw new Error('Request failed');
            }
        },
        SetTimeout: function (callback: () => void, timeout: number | undefined) {
            return setTimeout(callback, timeout);
        },
        SetInterval: function (callback: () => void, interval: number | undefined) {
            return setInterval(callback, interval);
        },
        ClearTimeout: function (timeout: string | number | NodeJS.Timeout | undefined) {
            clearTimeout(timeout);
        },
        ClearInterval: function (interval: string | number | NodeJS.Timeout | undefined) {
            clearInterval(interval);
        }
};

const nSdkInstance = new BsdkInstance(
        nsdkConfig.app_id,
        nsdkConfig.instance_name,
        {
            appName: 'BSDK RN Sample App',
            deviceId: 'testDeviceId',
            nol_sdkDebug: 'debug', // remove debug flag when going to production
            domlessEnv: // "1" for React Native | "2" for Amazon | "3" for NodeJS | "4" for Custom
            // reference SDK interface documentation
            // for additonal metadata properties
        },
		implementationHooks
);

// Sample VideoPlayer component
const VideoPlayer = (props) => {
    /**
     * Implementation of video player component will vary across the board, for Nielsen DOM-less SDK integration
     * clients need only setup event listeners with corresponding ggPM() calls.
     *
     * Please refer to chosen video player documentation on available events
    */

    const video = useRef<Video>(null);
    let previousPlayhead = 0; // keep track of previous playhead position
    let metadataLoaded = false; // in case of replay scenario set flag for metadata load


    // Sample video metadata
    const videometadata = {
        'type': 'content',
        'assetid': 'VID-123456',
        'program': 'program name',
        'title': 'episode title with season and episode number',
        'length': 'length in seconds',
        'airdate': '20210321 09:00:00',
        'isfullepisode': 'y',
        'adloadtype': '2',
        'segB': 'custom segment B', // optional
        'segC': 'custom segment C', // optional
        'crossId1': 'Standard Episode ID', // optional
        'crossId2': 'Content Originator' //optional
    }

    const setUpEventListeners = (): void => {
        video.addEventListener('ended', onEnded);
        video.addEventListener('timeupdate', onTimeUpdate);
        video.addEventListener('playing', onPlay);
        video.addEventListener('metadata', onMetadata )
    };

    const onEnded = () => {
        // Nielsen SDK ggPM 'end' event
        if (nSdkInstance) {
        nSdkInstance.then((instance: any) => {
            instance.ggPM('end', Math.round(video.currentTime));
            metadataLoaded = false;
        });
        }
    };

    const onTimeUpdate = () => {
        const currPlayhead = Math.round(video.current?.currentTime!);
        if (currPlayhead > 0 && currPlayhead !== previousPlayhead) {
            previousPlayhead = currPlayhead;
            // Nielsen SDK ggPM 'setplayheadposition' event
            if (nSdkInstance) {
                nSdkInstance.then((instance: any) => {
                    instance.ggPM('setplayheadposition', currPlayhead);
                });
            }
        }
    };

    // NOTE: some players may have an event when video metadata is loaded, recommended to use if available. E.g. loadedmetadata
    const onPlay = () => {
        // Nielsen SDK ggPM 'loadmetadata' event
        if (nSdkInstance && !metadataLoaded) {
            nSdkInstance.then((instance: any) => {
                instance.ggPM('loadmetadata', videometadata);
                metadataLoaded = true;
            });
        }
    };

    // NOTE: retireval of ID3 tags will vary by player and is primarily used for DTVR measurement
    const onMetadata = (data) = {
        if (data.metadata && data.metadata.key === 'PRIV') {
            nSdkInstance.then(instance => {
            instance.ggPM('sendid3', data.metadata.info);
            });
        }
    };

    return (
        <View>
            <Video
                source={{ uri: 'https://www.w3schools.com/html/mov_bbb.mp4' }}
            />
        </View>
    );
};

```

## Release Notes

### April 22, 2025

#### DOM-less SDK version 1.2.0

- Added @types folder in package for TypeScript related errors.

### Octorber 30, 2024

#### DOM-less SDK version 1.1.0

- Support for Nielsen Digital Content Ratings (DCR) static product.
- Support to cache pings when network is not available.

### July 31, 2024

#### DOM-less SDK version 1.0.0

- Support for Nielsen Digital Content Ratings (DCR) video product
- Support for Nielsen Digital TV Ratings (DTVR) product
- Support for various domless environments like NodeJS, ReactNative etc.
- Added features like First Party ID, Optout, collections of persistent identifiers like hashed email and UID2, collection of deviceID etc.
- Support SDK multi-instance

### June 13, 2024

#### DOM-less SDK version 0.0.7

- Support for capturing user optout during SDK initialization and/or configuration file
- Pass in User-Agent string as part of headers in Fetch impelementation hook (UA passed in by client)
- Support for capturing hashed email through SDK initialization
- Support of Storage implementation hook and will be used first party id
- `.ok()` option for checking SDK initialization
