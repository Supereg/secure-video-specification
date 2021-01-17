# HomeKit Secure Video Unofficial Specification - R2

## 1. Overview

### Service Definitions

#### Cameras

HomeKit Secure-Video cameras need to expose the same services as normal cameras with the following changes: 

* Every `RTPStreamManagement` service must add the `Active` characteristic
(This is used to indicate that the camera is fully turned off)
* The required amount of `RTPStreamManagement` services was dropped from two to one.
* The `MotionSensor` service is required (to indicate movement and thus a start and stop of a recording)
* The `CameraOperatingMode` service is required
* The `DataStreamManagement` service is required (to initiate HomeKit Data Stream communication)
* The `CameraEventRecordingManagement` service is required. It needs to link to the `MotionSensor` and `DataStreamManagement` service

If `MotionSensor` or `OccupancySensor` are added, they must expose the `Active` characteristic.

#### Doorbells

HomeKit Secure-Video doorbells need to expose the same services as both doorbells and secure-video cameras.

### Active states

Every Secure-Video enabled camera can be set to four different states: `Off`, `Detect Activity`, `Stream`
and `Stream & Allow Recording`.  
Depending on the state the following `Active` characteristics for the given services are set.

| Camera-States            | RTPStreamManagement `Active` | CameraOperatingMode `HomeKitCameraActive` | CameraEventRecordingManagement `Active` | 
| :----------------------: | :---: | :---: | :---: |
| Off                      | false | false | false |
| Detect Activity          | false  | true | false |
| Stream                   | true  | true  | false |
| Stream & Allow Recording | true  | true  | true  |

### Recording requirements

* The camera needs so support basic motion detection
* The camera needs to support encoding, buffering and transmitting of fragmented MP4 - Pretty similar to Section 3.3 
[RFC 8216](https://tools.ietf.org/html/rfc8216#section-3.3) (HLS). The RFC refers to
[ISO/IEC 14496-12 ISO base media file format](https://mpeg.chiariglione.org/standards/mpeg-4/iso-base-media-file-format/text-isoiec-14496-12-5th-edition).  
How exactly the data is transmitted can be read
in the [HDS Packet Formats](#4-homekit-data-stream-packet-formats) section and the [Flow of events](#6-flow-of-events)
section.
* Supported video codecs are: h.264, h.265 (possibly)
* Supported audio codecs are: AAC-LC, AAC-ELD
* Typical resolutions one might support (a camera might support more):
    * 640x480
    * 1024x768
    * 1280x960
    * 1600x1200
    * 2048x1536
    * 640x360
    * 1280x720 (Mandatory)
    * 1920x1080 (Mandatory, default at 24 or 30 fps)
    * 3840x2160
* Frame rates:
    * 15 fps (Mandatory)
    * 24 fps
    * 30 fps (One of 24 fps or 30 fps is mandatory)


## 2. Services

### 2.1 CameraOperatingMode

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 0000021A-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.service.camera-operating-mode |
| Required Characteristics  | [3.2 EventSnapshotActive](#32-eventsnapshotsactive) <br> [3.3 HomeKitCameraActive](#33-homekitcameraactive) <br> [3.6 PeriodicSnapshotsActive](#36-periodicsnapshotsactive) |
| Optional Characteristics  | [3.4 ManuallyDisabled](#34-manuallydisabled) <br> [3.5 NightVision](#35-nightvision) <br> [3.12 ThirdPartyCameraActive](#312-thirdpartycameraactive) <br> [3.1 CameraOperatingModeIndicator](#31-cameraoperatingmodeindicator) <br> [3.13 Diagonal Field of View](#313-diagonalfieldofview) |

### 2.2 CameraEventRecordingManagement

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000204-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.service.camera-recording-management |
| Required Characteristics  | Active <br> [3.8 SupportedCameraRecordingConfiguration](#38-supportedcamerarecordingconfiguration) <br> [3.9 SupportedVideoRecordingConfiguration](#39-supportedvideorecordingconfiguration) <br> [3.10 SupportedAudioRecordingConfiguration](#310-supportedaudiorecordingconfiguration) <br> [3.11 SelectedCameraRecordingConfiguration](#311-selectedcamerarecordingconfiguration) <br> [3.7 RecordingAudioActive](#37-recordingaudioactive) |

## 3. Characteristics

### 3.1 CameraOperatingModeIndicator

This characteristic indicates if the camera LED, which shows the current state of the camera (see [states](#active-states)),
should be turned on. Controlled by "Camera status light" setting in the Home App.

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 0000021D-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.camera-operating-mode-indicator |
| Permissions               | Paired Read, Paired Write, Notify, Timed Write |
| Format                    | bool |
| Valid Values              | 0 - "Hardware LED is disabled" <br> 1 - "Hardware LED is enabled" |

### 3.2 EventSnapshotsActive

This characteristic indicates if the option _"Camera Settings" -> Notifications -> "Allow Snapshots in Notifications"_
is turned on. If this option is turned on, a notification from this camera sent to anyone in this home, will include
a snapshot of the motion or activity.

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000223-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.event-snapshots-active |
| Permissions               | Paired Read, Paired Write, Notify, Timed Write |
| Format                    | bool |
| Valid Values              | 0 - "Snapshots in notifications are turned off" <br> 1 - "Snapshots in notifications are turned on" |

### 3.3 HomeKitCameraActive

This characteristic indicates if the camera should detect activity (Unsure if activity just means motion detection or 
also button presses for doorbell accessories).

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 0000021B-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.homekit-camera-active |
| Permissions               | Paired Read, Paired Write, Notify, Timed Write |
| Format                    | bool |
| Valid Values              | 0 - "Activity detection should not be enabled" <br> 1 - "Activity detection should be enabled" |

### 3.4 ManuallyDisabled

This characteristic indicates if the camera was manually turned off, for example using a physical switch on the camera.

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 0000021B-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.manually-disabled |
| Permissions               | Paired Read, Notify |
| Format                    | bool |
| Valid Values              | 0 - "Camera is not manually disabled" <br> 1 - "Camera was manually disabled" |

### 3.5 NightVision

_This characteristic is already present in the current HAP spec_
This characteristic indicates if automatic night vision should be turned on.

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 0000011B-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.night-vision |
| Permissions               | Paired Read, Paired Write, Notify |
| Format                    | bool |
| Valid Values              | 0 - "Disable night-vision mode" <br> 1 - "Enable night-vision mode" |

### 3.6 PeriodicSnapshotsActive

Exact behaviour unclear. Seems to be always set to `true` regardless of anyone viewing periodic snapshots or not.

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000225-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.periodic-snapshots-active |
| Permissions               | Paired Read, Paired Write, Notify, Timed Write |
| Format                    | bool |

### 3.7 RecordingAudioActive

This characteristic indicates if recordings should include audio.

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000226-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.recording-audio-active |
| Permissions               | Paired Read, Paired Write, Notify, Timed Write |
| Format                    | uint8 |
| Valid Values              | 0 - "Audio should not be included in recordings" <br> 1 - "Audio recording is active" |

### 3.8 SupportedCameraRecordingConfiguration

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000204-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.supported-camera-recording-configuration |
| Permissions               | Paired Read, Notify |
| Format                    | tlv8 |

A read request on this characteristic returns the following structure:

##### Supported Camera Recording Configuration:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Prebuffer length | 1 | Size of the prebuffer in milliseconds. Must be a multiple of the fragment length.<br>(typically 4000-8000ms) |
| 2 | Event Trigger Options | 8 | Bitmask of trigger types: <br>  0x01 - Motion <br> 0x02 - Doorbell |
| 3 | Media Container Configurations | N | List of supported media container configurations. <br>Most cameras out there do only expose one entry. |

##### Media Container Configuration:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Media Container Type | 1 | Container types: <br> 0 - Fragmented MP4 |
| 2 | Media Container Parameters | N | Media container parameters |

##### Media Container Parameters:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Fragment Length | 4 | Length of one mp4 fragment in milliseconds<br>(typically 4000ms) |

### 3.9 SupportedVideoRecordingConfiguration

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000206-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.supported-video-recording-configuration |
| Permissions               | Paired Read, Notify |
| Format                    | tlv8 |

The value of this characteristic is a TLV8-encoded list of supported video codecs:

##### Supported Video Recording Configuration:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Codec Configuration | N | Codec information and the configurations supported for the codec<br> There is one TLV of this type per supported codec |

##### Video Codec Configuration:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Codec | 1 | Type of video codec:<br>0 - H.264 <br>1 - H-265 |
| 2 | Video Codec Parameters | N | Video Codec specific parameters |
| 3 | Video Attributes | N | Video Attributes supported for the codec (tlv list) |

Video Codec Configuration TLV contains exact one tlv of 'Video Codec Parameters' and one entry of
'Video Attributes' per supported resolution/frame rate combination. 

##### Video Codec Parameters:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | ProfileID | 1 | List of supported H.264 profiles (tlv list is separated by empty tlvs): <br>0 - Baseline Profile <br>1 - Main Profile <br>2 - High Profile |
| 2 | Level | 1 | List of supported H.264 levels (tlv list is separated by empty tlvs): <br>0 - 3.1 <br>1 - 3.2 <br>2 - 4 |
| 3 | Bitrate | 4 | Only present in the Selected Camera Recording Configuration request: <br>Selected video bitrate. Typically, secure video requests 2000kbps when face recognition is enabled, and 800kbps otherwise.   |
| 4 | iFrame_Interval | 4 | Only present in the Selected Camera Recording Configuration request: <br>Selected key frame interval in milliseconds. Typically 4000ms. Seems to be the same value as the fragment length. So every mp4 fragment MUST begin with a keyframe |

##### Video Codec Attributes:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Image width | 2 | Image width in pixels |
| 2 | Image height | 2 | Image height in pixels |
| 3 | Frame rate | 1 | Maximum frame rate |

### 3.10 SupportedAudioRecordingConfiguration

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000207-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.supported-audio-recording-configuration |
| Permissions               | Paired Read, Notify |
| Format                    | tlv8 |

The value of this characteristic is a TLV8-encoded list of supported audio codecs:

##### Supported Audio Recording Configuration:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Codec Configuration | N | Codec information and the configurations supported for the codec<br> There is one TLV of this type per supported codec |

##### Audio Codec Configuration:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Codec | 1 | Type of audio codec:<br>0 - AAC-LC <br>1 - AAC-ELD |
| 2 | Audio Codec Parameters | N | Video Codec specific parameters |

##### Audio Codec Parameters:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Channels | 1 | Count of audio channels |
| 2 | Bitrate Modes | 1 | List (probably empty tlv separated?) of supported audio bitrate modes: <br>0 - Variable <br>1 - Constant |
| 3 | Sample rates | 1 | List (probably empty tlv separated?) of supported sample rates: <br>0 - 8 kHz <br>1 - 16 kHz <br>2 - 24 kHz <br>3 - 32 kHz <br>4 - 44.1 kHz <br>5 - 48 kHz|
| 4 | Max Audio Bitrate | 4 | Only present in the Selected Camera Recording Configuration request: <br>maximum selected audio bitrate |

### 3.11 SelectedCameraRecordingConfiguration

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000209-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.selected-camera-recording-configuration |
| Permissions               | Paired Read, Paired Write, Notify |
| Format                    | tlv8 |

The structure of the write value looks like the following:

##### Selected Camera Recording Configuration:

| Type | Name | Format | Description |
| --- | --- | --- | --- |
| 1 | Selected General Configuration | N | The selected [recording configuration](#supported-camera-recording-configuration) |
| 2 | Selected Video Configuration | N | The selected [video recording configuration](#supported-video-recording-configuration) |
| 3 | Selected Audio Configuration | N | The selected [audio recording configuration](#supported-audio-recording-configuration) |

### 3.12 ThirdPartyCameraActive

_Usage and behaviour of this characteristic is currently pretty unclear._

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 0000021C-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.third-party-camera-active |
| Permissions               | Paired Read, Notify |
| Format                    | bool |

### 3.13 DiagonalFieldOfView

| Property                  | Value                                |
| ------------------------- | ------------------------------------ |
| UUID                      | 00000224-0000-1000-8000-0026BB765291 |
| Type                      | public.hap.characteristics.diagonal-fov |
| Permissions               | Paired Read, Notify |
| Format                    | float |
| Minimum Value             | 0 |
| Maximum Value             | 360 |
| Unit                      | arcdegrees |

## 4. HomeKit Data Stream Packet Formats
### 4.1 Start

When the camera detects motion it will send a hap event for the characteristic as usual.
After that, one of the connected Home Hubs will send an open request.

The header should use `dataSend` as the protocol and `open` as the topic.  
The **request** has the following message fields:

| Key | Type | Description |
| --- | ---- | ----------- |
| target | string | `home hub` - to signify the direction of the send |
| type | string | `ipcamera.recording` - the type of the stream |
| streamId | int | used to identify this stream; chosen by the home hub |

The **response** has the following message fields:

| Key | Type | Description |
| --- | ---- | ----------- |
| status | int | Indicates if stream could be opened. Available codes are unknown: <br> 0 - Success |

### 4.2 Binary Data

The header should use `dataSend` as the protocol and `data` as the topic.
The **event** has the following message fields:

| Key | Type | Description |
| --- | ---- | ----------- |
| streamId | int | Same identifier used in the dataSend.open |
| packets | array | Array of dictionaries. Usually length = 1 |

A packet dictionary looks like the following:

| Key | Type | Description |
| --- | ---- | ----------- |
| data | bytes | Packet data |
| metadata | dictionary | Metadata for the packet |

Metadata for recording chunks is defined as:

| Key | Type | Description |
| --- | ---- | ----------- |
| dataType | string | `mediaInitialization` - for the first event message which contains mp4 initializing `ftyp` and `moof` boxes <br> `mediaFragment` - for all other packets which contains fragmented mp4 segments |
| dataSequenceNumber | int | Starting by `1` (with the mediaInitialization packet) and incrementing for every mp4 segment |
| dataChunkSequenceNumber | int | Starting by `1`; enumerates every data chunk of a mp4 segment (if a mp4 segment is to big it can be split in multiple packets using this chunk number) |
| isLastDataChunk | boolean | `true` when the data chunk is the last for the current sequence/mp4 segment |

### 4.3 Close

This event closes the stream and is sent by the home hub once the motion sensor is set back to "No motion detected"
(It seems that the home hub still waits for the last mp4 segment to be sent).  
The header should use `dataSend` as the protocol and `close` as the topic.

The **event** has the following message fields:

| Key | Type | Description |
| --- | ---- | ----------- |
| streamId | int | Same identifier used in the dataSend.open |
| reason | int | Example reasons: <br> 0 - Normal - Normal Close <br> 1 - Not Allowed - Home hub will not allow the Accessory to send this transfer <br> 2 - Busy - Home hub cannot accept this transfer right now <br> 3 - Cancelled - Accessory will not finish the transfer <br> 4 - Unsupported - Home hub does not support this stream type <br> 5 - Unexpected Failure - Some other protocol error occurred and the stream has failed <br> 6 - Timeout - Accessory could not start the session |

The accessory can also send this event message to indicate that the session errored unexpectedly and should be aborted.

## 5. Image Snapshots

The POST body of the `POST /resource` request received a new optional property `reason` with number type.

With the `reason` property a controller indicates the reason for a snapshot request:
* `0`: Request is the result of a periodic snapshot request.
* `1`: Request is the result of an event snapshot (e.g. to display image for a motion event).

If the accessory has [PeriodicSnapshotsActive](#36-periodicsnapshotsactive) turned off, any snapshot request without 
a `reason` property or the `reason` property set to `0` must be rejected.

If the accessory has [EventSnapshotsActive](#32-eventsnapshotsactive) turned off, any snapshot request without
a `reason` property or the `reason` property set to `1` must be rejected.


When rejecting a snapshot request the accessory must return `HTTP 207 Multi-Status` and
a HAP Status code of `-70401` (`INSUFFICIENT_PRIVILEGES`; if it was rejected due to the missing `reason` property)
or `-70412` (`NOT_ALLOWED_IN_CURRENT_STATE`, if the `reason` doesn't match the current set rules).

## 6. Flow of events

In this section I will give a brief overlook on how an activity will be recorded using secure-video.

* The secure-video camera gets paired.
* All available Home Hubs will open an HomeKit Data Stream Connection to the accessory
* The user sets the current [camera state](#active-states) to `Stream & Allow Recording`
* The configuration of the camera will be set up:
    * The accessory will receive a write request on the
      [SelectedCameraRecordingConfiguration](#311-selectedcamerarecordingconfiguration) characteristic
        * Important settings like the `prebuffer length` are configured
    * The `Active` characteristic of the [CameraEventRecordingManagement](#22-cameraeventrecordingmanagement) service will be
        set to true (and all other active characteristics getting updated according to the [camera state](#active-states))
* If the camera is set to detect motion it will continuously check the video stream for any movement as usual.
If recording is enabled, the camera will fill the pre buffer with mp4 fragments according to the First-In-Last-Out principle.
* If the camera detects motion (analogous for doorbell button presses) if will set the `Motion Detected` characteristic
of the `Motion Sensor` to true
    * After that, a home hub will initiate a bulk send session over HDS and sends a [`dataSend` `start` request](#41-start)
    with a new streamId.
    * The camera will now send a [mediaInitialization](#42-binary-data) `dataSend` `data` event with the below listed metadata.
    The mp4 data contains a `ftyp` and `moov` box.
        * `dataSequenceNumber`: 1
        * `dataChunkSequenceNumber`: 1
        * `isLastDataChunk`: true
    * After that the actual mp4 fragments are getting sent using [mediaFragment](#42-binary-data) `dataSend` `data` events.
    The mp4 data contains a `moof` and a `mdata` box and must start with a keyframe.
    The accessory will begin immediately by sending the fragments currently contained in the prebuffer (typically 2x4 seconds in length).
    After that the accessory will send any newly recorded mp4 fragments (typically 4s in length) when they become available
    (any fragment will be sent, where the recording was started while motion was still detected).  
    Every mp4 fragment gets a new incrementally assigned `dataSequenceNumber` (starting by 2 for the first segment in the preBuffer).
    If the size of one mp4 fragment is too big, it can be split into multiple chunks. Then every chunk is enumerated
    by the `dataChunkSequenceNumber`, while the last chunk must always be marked with `isLastDataChunk` equal to true.
    Current cameras seems to use a **maximum chunk size of 262,144 bytes** (or 0x40000 bytes).
* The HomeKit Home Hub receiving the mp4 fragments will analyze every mp4 fragment for moving objects, recognize faces, and decide,
according to the configured motion settings, if a given fragment will be flagged for motion or not.
The Home Hub will then assemble a recording from the flagged mp4 fragments and save it in iCloud. 
iCloud will then tell iPhones, iPads, HomePods and AppleTV to notify the user based on their notification settings.  
* When motion stops the accessory will set the `Motion Detected` characteristic  of the `Motion Sensor` to false.
The camera will still send out the last mp4 fragment which is currently recorded (remember: typically fixed 4s fragment length).
After a short time the Home Hub will send a [`dataSend` `close` event](#43-close) to indicate that the given 
transmission for the `streamId` is closed.

Example fmp4 files of a transmitted recording can be found in the [examples](./examples) directory.
Additionally, a full writeup of the transmitted HomeKit Data Stream payloads for the given example can be found 
[here](./examples/README.md).
