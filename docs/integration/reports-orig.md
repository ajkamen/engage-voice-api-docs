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
    * **Limit:** Integration reporting and aggregate-statistics requests are throttled per RingCX sub-account. Stay within both limits: **3 requests per 60 seconds** and **9 requests per 15 minutes**.
    * **Strategy:** Treat these as low-frequency reporting exports, not high-frequency polling endpoints. Cache results, request completed time windows, and use exponential backoff on `429 Too Many Requests` responses.

### Required Permissions & Scopes

For detailed instructions on obtaining your access token, please refer to the [RingCentral Authentication Guide](https://developers.ringcentral.com/engage/voice/guide/authentication/auth-ringcentral).

To successfully call the API, your RingCX account must be configured with **WEM Access**. If you lack the necessary permissions or receive an error when calling the API, please contact your Customer Success Manager (CSM)

---

## Interaction Metadata & Media

### Agent Segment Metadata

**Purpose:** Granular Interaction Auditing and Quality Management.

The **Interaction Metadata** API is used to reconstruct the complete "story" of a customer journey. While a standard report might show a single call, this API breaks that call down into specific **Segments**, representing every participant (Agent, IVR, queue, or Bot) involved.

A user would use this to perform forensic tracking or quality assurance. For example, if a customer was transferred three times, this API provides the metadata for all three segments, including specific timestamps for when each agent joined or left and whether recordings or transcripts are available for that specific portion of the call.

For new integrations, use the v2 endpoint:

```http
POST https://ringcx.ringcentral.com/voice/api/cx/integration/v2/accounts/{rcAccountId}/sub-accounts/{subAccountId}/interaction-metadata
Authorization: Bearer <ringcxAccessToken>
Content-Type: application/json
```

```json
{
  "segmentEndTime": "2026-03-02 09:00:00",
  "timeInterval": 1800,
  "timeZone": "US/Eastern"
}
```

Metadata availability follows the account's configured data retention period.

Set `segmentEndTime` to the beginning of the completed time window you want to query. The API searches forward by `timeInterval` seconds, up to 10800 seconds (3 hours).

The v2 response uses current customer-facing field names and includes voice interactions that end before an agent leg is created, such as IVR abandons, queue timeouts, and no-agent-available outcomes. Use `uii` as the interaction identifier, `ani` and `dnis` for caller and dialed addresses, `callType` for the interaction direction, and `agentId` for the RingCX agent identifier when an agent segment exists. The `dialogId` and `segmentId` fields remain the values needed to retrieve recordings, transcripts, and summaries. The `callResult` field is populated for voice interactions using RingCX Historical Analytics outcome semantics and is `null` for digital interactions.

| v1 field | v2 field | Notes |
| --- | --- | --- |
| `interactionId` | `uii` | Unique interaction identifier. |
| `dialogStartTimeMs` | `interactionStartTimeMs` | Interaction start time. |
| `dialogEndTimeMs` | `interactionEndTimeMs` | Interaction end time. |
| `dialogOrigination` | `callType` | Interaction direction, such as `INBOUND` or `OUTBOUND`. |
| `contactEndpointAddress` | `ani` | Caller address. |
| `channelEndpointAddress` | `dnis` | Dialed address. |
| `segmentParticipantId` | `agentId` | Populated when the segment has an agent. |
| N/A | `callResult` | Final voice call outcome, such as `Inbound Accepted` or `Inbound Abandoned`. |

```json
[
  {
    "uii": "202606020106368133110000000002",
    "dialogId": "s-v-23eb89e65275496496bc956876e45568-1780376796111",
    "segmentId": "p-v-23eb89e65275496496bc956876e45568-1780376796111-00e86b9fbcf05",
    "interactionStartTimeMs": "2026-06-02T05:06:36.000Z",
    "interactionEndTimeMs": "2026-06-02T05:07:21.000Z",
    "dialogDurationMs": 45000,
    "callType": "INBOUND",
    "channelType": "VOICE",
    "channelClass": "VOICE",
    "ani": "2098886424",
    "dnis": "9072901055",
    "agentId": null,
    "agentFullName": null,
    "productType": 3,
    "productId": "483",
    "termParty": "CALLER",
    "termReason": "HANGUP",
    "callResult": "Inbound Abandoned",
    "hasRecording": false,
    "hasTranscript": false
  }
]
```

Existing integrations that depend on the v1 field names can continue to use `/cx/integration/v1/.../interaction-metadata`.

* **Reference:** [Interaction Metadata API Details](https://developers.ringcentral.com/engage/voice/api-reference/Public-Integration-API/getInteractionMetadataV2)

### Retrieving Agent Segment Recordings & Transcripts

Once metadata is retrieved, users can access the specific media files.

* **Recordings:** Used for compliance archiving and quality evaluations. To account for processing time, allow at least 10 minutes after an interaction completes before retrieval.
* **Transcripts:** Used for text-based sentiment analysis and quick review of conversations without listening to audio. Like recordings, these should be accessed after the 10-15 minute processing window.

For more details on how to retrieve these files refer to the [Call Transcripts API](../analytics/reports/call-transcripts.md)

---

## Aggregated Statistics

### Queue Statistics (`agg-queue-stats`)

**Purpose:** Service Level Monitoring and Operational Health.

The **Queue Statistics** API provides a high-level view of how specific contact center queues are performing over defined time intervals (15, 30, 45, or 60 minutes). It aggregates data across the entire queue rather than focusing on individual agents.

Operations managers use this data to ensure the center is meeting its Service Level Agreements (SLAs). It is the primary tool for identifying volume trends, such as spikes in abandoned calls or excessive wait times, allowing for real-time or historical staffing adjustments.

* **Reference:** [Queue Statistics API Details](https://developers.ringcentral.com/engage/voice/api-reference/Public-Integration-API/buildAggQueueStats)

### Agent Statistics (`agg-agent-stats`)

**Purpose:** Workforce Productivity and Attendance Tracking.

The **Agent Statistics** API is designed to measure the general output and availability of the workforce. It focuses on how agents are spending their time across broad categories like "talking," "wrap-up," or "idle".

A user would use this API to generate daily or weekly productivity reports. It is commonly used for payroll verification and ensuring that agents are adhering to their scheduled shifts by comparing their active time against their expected hours.

* **Reference:** [Agent Statistics API Details](https://developers.ringcentral.com/engage/voice/api-reference/Public-Integration-API/buildAggAgentStats)

### Agent Extended Statistics (`agg-agent-extended-stats`)

**Purpose:** Deep-Dive Performance and Behavioral Analysis.

The **Agent Extended Statistics** API provides a more granular breakdown of performance metrics for agents across their assigned queues. It builds upon basic stats by adding specific handling details, such as transfer counts and precise handling times per queue.

This is the preferred tool for workforce planners and supervisors performing root-cause analysis. If a supervisor notices a drop in efficiency, they can use this API to determine if the issue is tied to a specific queue or if an agent is transferring a disproportionate number of interactions, indicating a potential need for further training.

* **Reference:** [Agent Extended Statistics API Details](https://developers.ringcentral.com/engage/voice/api-reference/Public-Integration-API/buildExtendedAggAgentStats)

---
