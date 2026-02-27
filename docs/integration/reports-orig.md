# Reports

The Reports APIs provide workforce management (WFM) tools to retrieve detailed agent interaction records and aggregated performance statistics across queues and agents. These endpoints are designed to feed external systems with raw data necessary for scheduling, compliance, and performance analysis.

## Strategic Overview

These tools allow developers to synchronize RingCX interaction data with third-party WFM platforms. By providing both granular segment metadata and high-level interval statistics, the API supports reporting requirements and forensic interaction tracking.

### Key Use Cases

* **WFM Synchronization:** Exporting agent activity and queue metrics to workforce management software for staffing optimization.
* **Compliance Archiving:** Maintaining a metadata trail of all interactions, including links to recordings and transcripts.
* **Performance Analysis:** Reviewing aggregated queue and agent data to identify trends in talk time and wrap-up durations.

### Real-Time vs. Latency Expectations

Data availability is subject to a propagation delay while interactions are finalized and indexed.

* **Data Availability:** Data must be fetched **at least 5 minutes** before the current time to avoid API errors.
* **Processing Buffer:** For interaction metadata and recordings, it is recommended to allow a 15-minute window for all media processing to complete.

!!! important "Rate Limiting & Stability"
    * **Limit:** Requests are limited to **4 calls per minute** per node.
    * **Strategy:** If the API returns a `429 Too Many Requests` status code, implement an **exponential backoff** strategy for subsequent retry attempts. This strategy involves doubling the delay after each consecutive 429 error (e.g., 1s, 2s, 4s...) rather than retrying at fixed intervals.

### Required Permissions & Scopes

For detailed instructions on obtaining your access token, please refer to the [RingCentral Authentication Guide](https://developers.ringcentral.com/engage/voice/guide/authentication/auth-ringcentral).

To successfully call the API, your RingCX account must be configured with **WEM Access**. If you lack the necessary permissions or receive an error when calling the API, please contact your Customer Success Manager (CSM).


---

## Interaction Metadata & Media

### Agent Segment Metadata

This endpoint retrieves metadata for both VOICE and DIGITAL channels within a specified time range. A **Dialog** represents the entire customer journey, while a **Segment** represents a specific participant's (Agent, IVR, or Bot) involvement in that journey.

`POST https://ringcx.ringcentral.com/voice/api/cx/integration/v2/admin/reports/accounts/{{subAccountId}}/interactionMetadata`

#### Request Body

| Parameter | Type | Requirement | Description |
| --- | --- | --- | --- |
| `segmentEndTime` | String | **Required** | The end of the logging interval (e.g., `2024-07-22 11:25:00`). |
| `timeInterval` | Integer | **Required** | Interval length in seconds. Maximum allowed length is 3600 (1 hour). |
| `timeZone` | String | **Required** | Timezone name used for report generation (e.g., `US/Eastern`). |

??? info "View Metadata Response Fields"


    | Field | Type | Description |
    | :--- | :--- | :--- |
    | `subAccountId` | Integer | The unique identifier for the RingCX sub-account. |
    | `interactionId` | String | Unique interaction ID used to connect different call segments together. |
    | `interactionStartTimeMs` | String | Start time of the interaction. Milliseconds precision. |
    | `interactionDurationMs` | Integer | The total duration of the interaction. Milliseconds precision. |
    | `segmentID` | Integer | Unique segment sequence ID within the interaction. |
    | `segmentAgentId` | Integer | RingCentral user ID of the agent. |
    | `segmentContactStartTimeMs` | String | Time agent joined the conversation. Milliseconds precision. |
    | `segmentContactEndTimeMs` | String | Time agent left the conversation. Milliseconds precision. |
    | `segmentDuration` | Integer | Segment duration in seconds. |
    | `interactionCallingAddress` | String | The ANI (caller ID) of the interaction. |
    | `interactionCalledAddress` | String | The DNIS (dialed number) of the interaction. |
    | `interactionDirection` | String | Direction of the interaction: `INBOUND` or `OUTBOUND`. |
    | `segmentRecordingURL` | String | URL for segment recording retrieval. |
    | `segmentAgentGroupId` | String | The ID of the agent group this agent belongs to for this segment. |
    | `segmentEvents` | Array | Ordered list of events (e.g., `REC_START`, `REC_STOP`). |
    | `campaignId` | String | Unique identifier for the campaign. |
    | `campaignName` | String | Name of the campaign. |
    | `dialMode` | String | Dial mode of the campaign (e.g., PREDICTIVE, PREVIEW). |
    | `hciType` | String | Human Call Initiator type. |
    | `dialType` | String | Type of dialing (e.g., AGENT_ORIGINATED). |
    | `systemDisposition` | String | Disposition Code set by the System. |
    | `agentDisposition` | String | Disposition Code set by the agent. |
    | `agentNotes` | String | Notes added by the agent. |
    | `ringSenseEnabled` | Boolean | Identifies if the recording is RingSense enabled. |


#### Python Example: Fetching Metadata

```python
import requests
import json

url = "https://ringcx.ringcentral.com/voice/api/cx/integration/v2/admin/reports/accounts/99999999/interactionMetadata"

payload = json.dumps({
  "segmentEndTime": "2024-07-22 11:25:00",
  "timeInterval": 1800,
  "timeZone": "US/Eastern"
})
headers = {
  'Content-Type': 'application/json',
  'Authorization': 'Bearer YOUR_ACCESS_TOKEN'
}

response = requests.post(url, headers=headers, data=payload)
print(response.json())


```

### Retrieving Agent Segment Recordings

!!!
    To retrieve agent call recordings, you must first ensure the agent segment recording feature is enabled for each queue or campaign you wish to monitor. Once enabled, the `interactionMetadata` API will indicate if a recording is available for a specific segment via the `segmentRecordingURL` field.

To account for processing time, please allow at least 10 minutes after an interaction completes before invoking this API to ensure the media is finalized and ready for retrieval.

`GET https://ringcx.ringcentral.com/voice/api/cx/integration/v1/accounts/{{rcAccountId}}/sub-accounts/{{subAccountId}}/recordings/dialogs/{{dialogId}}/segments/{{segmentId}}`

#### Request Parameters

| Parameter | Type | Requirement | Description |
| --- | --- | --- | --- |
| `rcAccountId` | String | **Required** | The unique identifier for your RingEX account. |
| `subAccountId` | String | **Required** | The unique identifier for your RingCX sub-account. |
| `dialogId` | String | **Required** | The unique ID for the interaction, obtained from metadata. |
| `segmentId` | String | **Required** | The unique ID for the specific segment, obtained from metadata. |

#### Special Note About Call Recording Formats

By default, call recordings are returned as a WAV file with PCM 16-bit encoding. To receive a compressed MP3 file instead, you must set a browser-like `User-Agent` in your request header.

| Key | Value |
| --- | --- |
| `User-Agent` | `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36` |

#### Python Example: Downloading a Recording

```python
import requests

# Parameters obtained from the Metadata API
url = "https://ringcx.ringcentral.com/voice/api/cx/integration/v1/accounts/980634004/sub-accounts/99999999/recordings/dialogs/d-123/segments/s-456"

headers = {
  'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
  'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36'
}

response = requests.get(url, headers=headers, stream=True)

if response.status_code == 200:
    with open("recording.mp3", 'wb') as f:
        for chunk in response.iter_content(chunk_size=1024):
            f.write(chunk)
    print("Recording downloaded successfully.")
else:
    print(f"Error: {response.status_code}")


```

### Retrieving Agent Segment Transcripts


This endpoint retrieves the transcribed text for a specific interaction segment. Like recordings, transcripts require processing time and should be accessed after the 10-15 minute processing window.

`GET https://ringcx.ringcentral.com/voice/api/cx/integration/v1/accounts/{{rcAccountId}}/sub-accounts/{{subAccountId}}/transcripts/dialogs/{{dialogId}}/segments/{{segmentId}}`

??? info "View Transcript Response Fields"


    | Field | Type | Description |
    | :--- | :--- | :--- |
    | `channelClass` | String | Classification of the channel: `VOICE` or `DIGITAL`. |
    | `transcript` | Array | Ordered list of transcribed messages. |
    | `participantId` | String | Unique ID of the participant who spoke. |
    | `participantType` | String | Type of participant: `agent` or `contact`. |
    | `timestamp` | String | The epoch timestamp of the specific message. |
    | `message` | String | The transcribed text content. |


#### Python Example: Fetching a Transcript

```python
import requests

url = "https://ringcx.ringcentral.com/voice/api/cx/integration/v1/accounts/980634004/sub-accounts/99999999/transcripts/dialogs/d-123/segments/s-456"

headers = {
  'Authorization': 'Bearer YOUR_ACCESS_TOKEN'
}

response = requests.get(url, headers=headers)

if response.status_code == 200:
    data = response.json()
    for entry in data.get('transcript', []):
        print(f"[{entry['participantType']}]: {entry['message']}")


```

---

## Aggregated Statistics

### Queue Statistics

This report returns performance metrics for all queues within a specified sub-account over a defined interval. Data is returned in JSON format and provides a high-level view of inbound traffic, abandoned interactions, and service level performance.

`POST https://ringcx.ringcentral.com/voice/api/cx/integration/v1/admin/reports/accounts/{{subAccountId}}/aggQueueStats`

#### Request Body

| Parameter | Type | Requirement | Description |
| --- | --- | --- | --- |
| `startDate` | String | **Required** | Start date and time in `YYYY-MM-DD HH:MM:SS` format. |
| `endDate` | String | Optional | End date and time in `YYYY-MM-DD HH:MM:SS` format. |
| `timeInterval` | Integer | **Required** | Interval length in minutes. Valid values: `15`, `30`, `45`, or `60`. |
| `timeZone` | String | **Required** | Timezone name used for report generation (e.g., `US/Eastern`). |
| `reportType` | String | Optional | Report type defining parameters (default: `WEM`). |
| `outputFormat` | String | **Required** | Format of the report: `CSV` or `XML`. |

??? info "View Queue Statistics Response Fields"


    | Field | Type | Description |
    | :--- | :--- | :--- |
    | `interval` | Integer | Period of time in minutes (15, 30, 45, or 60). |
    | `dateTimeFrom` | String | Start date for this specific interval. |
    | `queue` | Integer | The queue’s unique numeric identifier. |
    | `queueName` | String | Given name of the queue. |
    | `offDirectIxnCnt` | Integer | Inbound calls or chats where this queue was the original receiver. |
    | `overflowInIxnCnt` | Integer | Interactions where another queue was the original receiver (Overflow in). |
    | `abandIxnCnt` | Integer | Number of lost or abandoned interactions during this interval. |
    | `overflowOutIxnCnt` | Integer | Number of interactions sent to another queue (Overflow out). |
    | `answIxnCnt` | Integer | Number of answered interactions in this interval. |
    | `queuedAndAnswIxnDur` | Integer | Total queue time for all answered interactions (in seconds). |
    | `queuedAndAbandIxnDur` | Integer | Total queue time for all abandoned interactions (in seconds). |
    | `talkingIxnDur` | Integer | Total length of time in seconds for the interactions. |
    | `wrapUpDur` | Integer | Total length of time in seconds for agent disposition/after-call work. |
    | `queuedAnswLongestQueDur` | Integer | Longest time a caller waited in queue before being answered (seconds). |
    | `queuedAbandLongestQueDur` | Integer | Longest time a caller waited in queue before abandoning (seconds). |
    | `ansServicelevelCnt` | Integer | Number of interactions answered within the defined service level. |
    | `waitDur` | Integer | Total waiting time for agents ready and waiting on contacts (seconds). |
    | `abandShortIxnCnt` | Integer | Number of abandoned contacts with queue time less than 5 seconds. |
    | `abandWithinSlCnt` | Integer | Number of abandoned contacts within the defined service level. |
    | `completedContacts` | Integer | Total answered and wrapped-up contacts completed during the interval. |
    | `segmentHoldTime` | Integer | Total duration in milliseconds a customer was placed on hold (VOICE only). |


#### Python Example: Fetching Queue Stats

```python
import requests
import json

url = "https://ringcx.ringcentral.com/voice/api/cx/integration/v1/admin/reports/accounts/99999033/aggQueueStats"

payload = json.dumps({
  "startDate": "2024-03-04 00:00:00",
  "endDate": "2024-03-04 01:00:00",
  "timeInterval": 60,
  "timeZone": "US/Eastern",
  "outputFormat": "CSV"
})
headers = {
  'Content-Type': 'application/json',
  'Authorization': 'Bearer YOUR_ACCESS_TOKEN'
}

response = requests.post(url, headers=headers, data=payload)
print(response.json())


```

---

### Agent Extended Statistics Report

This report provides a granular breakdown of performance metrics for all agents across their assigned queues for a specific duration. It is primarily used to track individual agent productivity, including talk time and transfer counts.

`POST https://ringcx.ringcentral.com/voice/api/cx/integration/v1/admin/reports/accounts/{{subAccountId}}/aggAgentExtendedStats`

#### Request Body

| Parameter | Type | Requirement | Description |
| --- | --- | --- | --- |
| `startDate` | String | **Required** | Start date and time in `YYYY-MM-DD HH:MM:SS` format. |
| `endDate` | String | Optional | End date and time in `YYYY-MM-DD HH:MM:SS` format. |
| `timeInterval` | Integer | **Required** | Interval length in minutes (`15`, `30`, `45`, or `60`). |
| `timeZone` | String | **Required** | Timezone name used for report generation. |
| `reportType` | String | Optional | Report type defining parameters (default: `WEM`). |
| `outputFormat` | String | **Required** | Format of the report: `CSV` or `XML`. |

??? info "View Agent Extended Statistics Response Fields"


    | Field | Type | Description |
    | :--- | :--- | :--- |
    | `interval` | Integer | Period of time in minutes (15, 30, 45, or 60). |
    | `dateTimeFrom` | String | Start date for this specific interval. |
    | `agentId` | Integer | The agent's unique numeric identifier. |
    | `agentName` | String | The agent's full name. |
    | `queue` | Integer | The unique identifier for the queue the agent was active in. |
    | `queueName` | String | The name of the associated queue. |
    | `talkingIxnDur` | Integer | Total handling/talking time (in seconds) for the interval. |
    | `wrapUpDur` | Integer | Total wrap-up time (in seconds) associated with queue interactions. |
    | `answIxnCnt` | Integer | Number of interactions answered by the agent through the queue. |
    | `transferOutIxnCnt` | Integer | Number of interactions answered and then transferred to another queue. |


#### Python Example: Fetching Agent Extended Stats

```python
import requests
import json

url = "https://ringcx.ringcentral.com/voice/api/cx/integration/v1/admin/reports/accounts/99999033/aggAgentExtendedStats"

payload = json.dumps({
  "startDate": "2024-03-04 05:00:00",
  "endDate": "2024-03-04 06:00:00",
  "timeInterval": 60,
  "timeZone": "US/Eastern",
  "outputFormat": "CSV"
})
headers = {
  'Content-Type': 'application/json',
  'Authorization': 'Bearer YOUR_ACCESS_TOKEN'
}

response = requests.post(url, headers=headers, data=payload)
print(response.json())


```

---

## Implementation Strategy

### Recommended Polling Pattern (Sliding Window)

To maintain data integrity and account for the **5-minute propagation delay**, developers should utilize a "Sliding Window" polling strategy.

| Execution Time | Request Range | Purpose |
| --- | --- | --- |
| **10:20** | 10:00 – 10:15 | Captures all events finalized by 10:15. |
| **10:35** | 10:15 – 10:30 | Captures all events finalized by 10:30. |
| **10:50** | 10:30 – 10:45 | Captures all events finalized by 10:45. |

