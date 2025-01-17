# Getting the data

To get your meeting data, listen for POST requests to the webhook URL you set up in your account.\
There you'll get two types of data:

* **live meeting events**, as the bot interacts with the meeting. Right now, the events supported share bot status updates.\

* **post-meeting data:** the video-recording, speech time events, ...

### 1. Live Meeting Events

The events we send you during the meeting are in the following format:

```http
POST /your-endpoint HTTP/1.1
Host: your-company.com
Content-Type: application/json
x-spoke-api-key: YOUR-API-KEY

{
  "event": "bot.status_change",
  "data": {
    "bot_id": "asfdgfdewsrtiuydsfgklafsd",
    "status": {
      "code": "joining_call",
      "created_at": "2024-01-01T12:00:00.000Z"
    }
  }
}
```

Here we have:

* `event`: The key-value pair for bot status events. Always `bot.status_change`.\

* `data.bot_id`: The identifier of the bot.\

* `data.status.code`: The code of the event. One of:
  * `joining_call`: The bot has acknowledged the request to join the call.\

  * `in_waiting_room`: The bot is in the "waiting room" of the meeting.\

  * `in_call_not_recording`: The bot has joined the meeting, however it is not recording yet.\

  * `in_call_recording`: The bot is in the meeting and recording the audio and video.\

  * `call_ended`: The bot has left the call.\

* `data.status.created_at`: An ISO string of the datetime of the event.\




### 2. Success  webhook

\
To get the final meeting data, listen for POST requests to the webhook URL you set up in your account.&#x20;

You either get:

* &#x20; `"event": "complete",`\


or\


* &#x20; `"event": "failed".`\
  \


Here's an example for a `complete event`, assuming \`[https://your-company.com/your-endpoint](https://your-company.com/your-endpoint)\` is your webhook URL:

```http
POST /your-endpoint HTTP/1.1
Host: your-company.com
Content-Type: application/json
x-spoke-api-key: YOUR-API-KEY

{
  "event": "complete",
  "data": {
    "bot_id": "asfdgfdewsrtiuydsfgklafsd",
    "mp4": "https://bots-videos.s3.eu-west-3.amazonaws.com/path/to/video.mp4?X-Amz-Signature=...",
    "speakers": ["Alice", "Bob"],
    "transcript": [{
      "speaker": "Alice",
      "words": [{
        "start": 1.3348110430839002,
        "end": 1.4549110430839003,
        "word": "Hi"
      }, {
        "start": 1.4549110430839003,
        "end": 1.5750110430839004,
        "word": "Bob!"
      }]
    }, {
      "speaker": "Bob",
      "words": [{
        "start": 2.6583010430839,
        "end": 2.778401043083901,
        "word": "Hello"
      }, {
        "start": 2.778401043083901,
        "end": 2.9185110430839005,
        "word": "Alice!"
      }]
    }]
  }
}
```

Let's break this down:

* `bot_id`: the identifier of the bot.\

* `mp4`: A private AWS S3 URL of the mp4 recording of the meeting. Valid for one hour only.\

* `speakers`: The list of speakers in this meeting. Currently, this requires the transcription to be enabled. Stay tuned :sunny:\

* `transcript | optional`:  The meeting transcript. Only given when `speech_to_text` is set when asking for a bot. An array of:\

  * `transcript.speaker`: The speaker name.\

  * `transcript.words`: The list of words. An array of:
    * `transcript.words.start`: The start time of the word.
      * `transcript.words.end`: The end time of the word.
      * `transcript.words.word`: The word itself.





### 3.  Failed webhook

&#x20;Here's an example of a failed recording:

```http
POST /your-endpoint HTTP/1.1
Host: your-company.com
Content-Type: application/json
x-spoke-api-key: YOUR-API-KEY

{
  "event": "failed",
  "data": {
    "bot_id": bot_id,
    "error": "CannotJoinMeeting"
  }
}
```

The failure types can be:\


| Error                 | description                                                                                          |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| CannotJoinMeeting     | The bot could not join the meeting url provided.                                                     |
| TimeoutWaitingToStart | The bot has quitted after waiting to be accepted for `automatic_leave.waiting_room_timeout` seconds. |
| BotNotAccepted        | The bot has been refused in the meeting.                                                             |
| InternalError         | An unexpected error occurred, please contact us if the issue persist.                                |
| InvalidMeetingUrl     | The meeting\_url provided is not a valid (zoom, meet, teams) url.                                    |
