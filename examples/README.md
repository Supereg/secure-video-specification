# DataSend Recording Example

This example will demonstrate the file encoding and the datastream communication for 
one recording.

In the `chunks` folder are the individual data chunks per file in their given order as 
they would be transferred over HDS.

In the `fragments` folder the chunks got assembled to a mp4 fragment.

And the `fully-assembled.mp4` file represent the whole recording assembled to one file.

## DataStream `dataSend` communication

All HDS payloads below are encoded in json objects which HAP-NodeJS would also use to 
encode/decode those payloads

### open

Request (Controller -> Accessory):

```json
{
  "header": {
    "protocol": "dataSend",
    "request": "open",
    "id": 0
  },
  "message": {
    "type": "ipcamera.recording",
    "target": "controller",
    "streamId": 1
  } 
}
```

Response (Accessory -> Controller):

```json
{
  "header": {
    "protocol": "dataSend",
    "response": "open",
    "id": 0,
    "status": 0
  },
  "message": {
    "status": 0
  } 
}
```

### data transmission

#### Media Initialization

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "data"
  },
  "message": {
    "streamId": 1,
    "packets": {
      "metadata": {
        "dataType": "mediaInitialization",
        "dataSequenceNumber": 1,
        "isLastDataChunk": true,
        "dataChunkSequenceNumber": 1
      },
      "data": "contents of ./chunks/mediaInitialization.mp4"
    }
  } 
}
```

### Transmission of the Pre Buffer

#### Sequence 2

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "data"
  },
  "message": {
    "streamId": 1,
    "packets": {
      "metadata": {
        "dataType": "mediaFragment",
        "dataSequenceNumber": 2,
        "isLastDataChunk": false,
        "dataChunkSequenceNumber": 1
      },
      "data": "contents of ./chunks/sequence2_1.mp4"
    }
  } 
}
```

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "data"
  },
  "message": {
    "streamId": 1,
    "packets": {
      "metadata": {
        "dataType": "mediaFragment",
        "dataSequenceNumber": 2,
        "isLastDataChunk": true,
        "dataChunkSequenceNumber": 2
      },
      "data": "contents of ./chunks/sequence2_2.mp4"
    }
  } 
}
```

#### Sequence 3

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "data"
  },
  "message": {
    "streamId": 1,
    "packets": {
      "metadata": {
        "dataType": "mediaFragment",
        "dataSequenceNumber": 3,
        "isLastDataChunk": false,
        "dataChunkSequenceNumber": 1
      },
      "data": "contents of ./chunks/sequence3_1.mp4"
    }
  } 
}
```

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "data"
  },
  "message": {
    "streamId": 1,
    "packets": {
      "metadata": {
        "dataType": "mediaFragment",
        "dataSequenceNumber": 3,
        "isLastDataChunk": true,
        "dataChunkSequenceNumber": 2
      },
      "data": "contents of ./chunks/sequence3_2.mp4"
    }
  } 
}
```

### Transmission of recordings

#### Sequence 4

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "data"
  },
  "message": {
    "streamId": 1,
    "packets": {
      "metadata": {
        "dataType": "mediaFragment",
        "dataSequenceNumber": 4,
        "isLastDataChunk": false,
        "dataChunkSequenceNumber": 1
      },
      "data": "contents of ./chunks/sequence4_1.mp4"
    }
  } 
}
```

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "data"
  },
  "message": {
    "streamId": 1,
    "packets": {
      "metadata": {
        "dataType": "mediaFragment",
        "dataSequenceNumber": 4,
        "isLastDataChunk": true,
        "dataChunkSequenceNumber": 2
      },
      "data": "contents of ./chunks/sequence4_2.mp4"
    }
  } 
}
```

### close

```json
{
  "header": {
    "protocol": "dataSend",
    "event": "close"
  },
  "message": {
    "streamId": 1,
    "status": 0
  } 
}
```
