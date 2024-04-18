# Nielsen DOM-less SDK

## License
Nielsen SDK contains material that is protected by copyright laws, patent laws, trade secret laws, and by international treaty provisions and is Copyright © 2024 The Nielsen Company (US) LLC. All intellectual property rights and licenses therein are reserved by The Nielsen Company (US) LLC and its licensors. Please read the license agreement presented [here](https://engineeringportal.nielsen.com/docs/Special:ClickThrough), which must be accepted in order to download the Nielsen SDKs.
For more information, reach out to your Nielsen Technical Account Manager(TAM).

## Overview
This project is an API that allows our clients to integrate the Nielsen SDK in DOM-less environments, e.g., React Native, Node, etc.

## Installation
Install with `npm install https://github.com/NielsenDigitalSDK/bsdk-domless#develop` and import the `BsdkInstance` into video player component

`import { BsdkInstance } from 'bsdk-domless'`

## Exposed Interface
The exposed interface is as follows:
1. `ggPM` - method to send messages to the Nielsen SDK
2. `processEvent` _(still in progress)_ - method to send app state to the Nielsen SDK, e.g., Focus, Blur, AppClose, etc.

### Events that can be passed to `ggPM` method
1. `loadmetadata` - This event is used to send DCR Video metadata to the Nielsen SDK. This event should be called when the video metadata is loaded.
```javascript
    instance.ggPM('loadmetadata', {
        'type': 'content',
        'length': '300',
        'censuscategory': 'Enlisted',
        'title': 'Channel1',
        'assetid': '204558915991',
        'section': 'ProgramAsset8',
        'tv': 'true',
        'adModel': '0',
        'dataSrc': 'cms'
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
4. `pause` - This event is used to send the pause event to the Nielsen SDK. This event should be called when the video is paused, passing the playhead at that time.
```javascript
    instance.ggPM('pause', 15);
```
5. `play` - This event is used to send the play event to the Nielsen SDK. This event should be called when the video is played; often used when resuming from pause, passing the playhead at that time.
```javascript
    instance.ggPM('play', 30);
```
6. `stop` - This event is used to send the stop event to the Nielsen SDK. This event should be called when transitioning from ads to content and content to ads, passing the playhead at that time.
```javascript
    instance.ggPM('stop', 120);
```
7. `sendid3` - This event is used to send the id3 event to the Nielsen SDK. This event should be called when the id3 event is triggered, passing the id3 data from the stream.
```javascript
    instance.ggPM('sendid3', '<id3 metadata received>');
```

## Sample DCR Video Integration
```javascript
import { BsdkInstance } from 'bsdk-domless';

const nsdkConfig = {
    app_id: 'DHG163HR-XXXX-XXXX-XXXX-XXXXXXXXXXXX',
    instance_name: 'videoInstance',
};

const bsdk = new BsdkInstance(
        nsdkConfig.app_id,
        nsdkConfig.instance_name,
        {
            appName: 'BSDK RN Sample App',
            deviceId: 'testDeviceId',
            nol_sdkDebug: 'debug',
            // reference SDK interface documentation
            // for additonal metadata properties
        }
);

// Sample VideoPlayer component
const VideoPlayer = (props) => {
    // Sample video metadata could be something as follows:
    const videometadata = {
        'type': 'content',
        'length': '300',
        'censuscategory': 'Enlisted',
        'title': 'Channel1',
        'assetid': '204558915991',
        'section': 'ProgramAsset8',
        'tv': 'true',
        'adModel': '0',
        'dataSrc': 'cms'
    }

    let previousPlayhead = 0;
    let playhead;

   // Load video metadata when video loads and prior to sending playhead positions
    const handlePlaybackLoad = () => {
        if(bsdk) {
            bsdk.then(instance => {
                instance.ggPM('loadmetadata', videometadata);
            });
        }
    };

    // Send the current playhead position to the Nielsen SDK
    const handlePlaybackProgress = (data) => {
        let currentTime = data.currentTime;
        playhead = Math.round(data.currentTime);

        if (playhead !== previousPlayhead && playhead > 0) {
            previousPlayhead = playhead;
            // BSDK setPlayheadPosition
            if(bsdk) {
                bsdk.then(instance => {
                    instance.ggPM('setplayheadposition', playhead);
                });
            }
        }
    };

    // Send end event to Nielsen SDK
    const handlePlaybackEnd = () => {
        if (bsdk) {
            bsdk.then(instance => {
                instance.ggPM('end', playhead);
            });
        }
    };

    return (
        <View>
            <Video
                source={{ uri: 'https://www.w3schools.com/html/mov_bbb.mp4' }}
                onPlaybackLoad={handlePlaybackLoad}
                onProgress={handlePlaybackProgress}
                onEnd={handlePlaybackEnd}
            />
        </View>
    );
};

```
