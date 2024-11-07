# Sending a bot



You can summon a bot:&#x20;

1. Immediately to a meeting, provided your bot pool is sufficient
2. Or reserve one to come in 4 minutes.

Here's an example POST request to [https://api.meetingbaas.com/bots](https://api.meetingbaas.com/bots), sending a bot to a meeting:

{% tabs %}
{% tab title="Bash" %}
{% code title="join_meeting.sh" fullWidth="true" %}
```bash
curl -X POST "https://api.meetingbaas.com/bots" \
     -H "Content-Type: application/json" \
     -H "x-meeting-baas-api-key: YOUR-API-KEY" \
     -d '{
           "meeting_url": YOUR-MEETING-URL,
           "bot_name": "Ai notetaker",
           "recording_mode": "speaker_view",
           "bot_image": "https://company.org/bot.jpg",
           "entry_message": "I am a good meeting bot :)",
           "reserved": false,
           "speech_to_text": {
             "provider": "Default"
           },
           "automatic_leave": {
             "waiting_room_timeout": 600
           }
         }'
```
{% endcode %}
{% endtab %}

{% tab title="Python" %}
{% code title="join_meeting.py" %}
```python
import requests
url = "https://api.meetingbaas.com/bots"
headers = {
    "Content-Type": "application/json",
    "x-spoke-api-key": YOUR-API-KEY,
}
config = {
    "meeting_url": YOUR-MEETING-URL,
    "bot_name": "Ai notetaker",
    "recording_mode": "speaker_view",
    "bot_image": "https://default.org/bot.jpg",
    "entry_message": "I am a good meeting bot :)",
    "reserved": False,
    "speech_to_text": {
        "provider": "Default"
    },
    "automatic_leave": {
        "waiting_room_timeout": 600  # 10 minutes in seconds
    }
}
response = requests.post(url, json=config, headers=headers)
print(response.json())
```
{% endcode %}
{% endtab %}

{% tab title="javascript" %}
```javascript
fetch('https://api.meetingbaas.com/bots', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'x-spoke-api-key': YOUR-API-KEY
    },
    body: JSON.stringify({
        meeting_url: YOUR-MEETING-URL,
        bot_name: 'Ai notetaker',
        reserved: false,
        recording_mode: "speaker_view",
        bot_image: 'https://example.com/image.png',
        entry_message: 'I am a good meeting bot :)',
        speech_to_text: {
            provider: 'Default'
        },
        automatic_leave: {
            waiting_room_timeout: 900
        }
    })
})
.then(response => response.json())
.then(data => console.log(data.bot_id))
.catch(error => console.error('Error:', error));
```
{% endtab %}
{% endtabs %}

Let's break this down:

* `meeting_url`: The meeting URL to join. Accepts Google Meet, Microsoft Teams or Zoom URLs. (Required)
* `bot_name`: The display name of the bot. (Required)
* `reserved`: (Required)
  * `false`: Sends a bot from our pool of _always ready_ meeting bots, immediately. Beware that **demand might temporarily be higher than the number of currently available bots**, which could imply a delay in the bot joining. When possible, prefer the true option which reserves an instance of an AI meeting bot.
  * `true`: Reserves in advance a meeting bot for an upcoming meeting, ensuring the presence of the bot at the start of the meeting, typically for planned calendar events. You need to **call this route exactly 4 minutes before the start of the meeting**.
* `bot_image: string | null`: The URL of the image the bot will display. Must be a valid URI format. Optional.
* `recording_mode`: Optional. One of:
  * `'speaker_view'`: (default) The recording will only show the person speaking at any time
  * `'gallery_view'`: The recording will show all the speakers
  * `'audio_only'`: The recording will be a mp3
* `entry_message`: Optional. The message the bot will write within 15 seconds after being accepted in the meeting.
* `speech_to_text`: Optional. If not provided, no transcription will be generated and processing time will be faster.
  * Must be an object with:
    * `provider`: One of:
      * `"Default"`: Standard transcription, no API key needed
      * `"Gladia"` or `"Runpod"`: Requires their respective API key to be provided
    * `api_key`: Required when using Gladia or Runpod providers. Must be a valid API key from the respective service.
* `automatic_leave`: Optional object containing:
  * `waiting_room_timeout`: Time in seconds the bot will wait in a meeting room before dropping. Default is 900 (15 minutes)
  * `noone_joined_timeout`: Time in seconds the bot will wait if no one joins the meeting

Additional optional parameters:

* `webhook_url`: URL for webhook notifications
* `deduplication_key`: String for deduplication. By default, Meeting Baas will reject you sending multiple bots to a same meeting within 5 minutes, to avoid spamming.&#x20;
* `streaming`: Object containing optional `input` and `output` streaming endpoints
* `extra`: Additional custom data
* `start_time`: UTC timestamp (uint64) for scheduled start time

This request will respond with the identifier of the bot you just created:

```http
HTTP/2 200
Content-Type: application/json

{
  "bot_id": 42
}
```


