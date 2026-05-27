# Media Control APIs

The Media Control APIs let you start and stop audio streams for a dialog or a specific segment. Use them with call streaming integrations when media should be captured selectively instead of streamed for every eligible interaction.

## Strategic Overview

Call streaming profiles define where audio can be streamed. Media control endpoints let an integration decide when to start or stop streaming for a specific interaction after it has identified the dialog or segment that should be processed.

### Key Use Cases

* **Selective Live Analytics:** Start a stream only when a workflow determines that live analysis is needed.
* **Compliance Control:** Stop streaming before sensitive information is discussed.
* **Targeted Recording Pipelines:** Stream only selected segments to external storage or AI services.
* **Troubleshooting:** Start a temporary stream for a specific dialog during an investigation.

### Required Permissions & Scopes

Your application needs the `ReadAccounts` OAuth scope and the account must be configured for call streaming. See [Call Streaming Getting Started](getting-started.md) for profile setup.

## Dialog-Level Streams

Dialog-level streams apply to the full interaction dialog.

| Operation | Method and Path | API Reference |
| --- | --- | --- |
| Start dialog stream | `POST /voice/api/cx/integration/v1/accounts/{rcAccountId}/sub-accounts/{subAccountId}/dialogs/{dialogId}/streams` | [Reference](https://developers.ringcentral.com/engage/voice/api-reference/Media-Control/startDialogStream) |
| Stop dialog stream | `DELETE /voice/api/cx/integration/v1/accounts/{rcAccountId}/sub-accounts/{subAccountId}/dialogs/{dialogId}/streams/{streamId}` | [Reference](https://developers.ringcentral.com/engage/voice/api-reference/Media-Control/stopDialogStream) |

## Segment-Level Streams

Segment-level streams apply to a specific segment within the dialog. Use segment-level streams when only one agent leg or participant segment should be streamed.

| Operation | Method and Path | API Reference |
| --- | --- | --- |
| Start segment stream | `POST /voice/api/cx/integration/v1/accounts/{rcAccountId}/sub-accounts/{subAccountId}/dialogs/{dialogId}/segments/{segmentId}/streams` | [Reference](https://developers.ringcentral.com/engage/voice/api-reference/Media-Control/startSegmentStream) |
| Stop segment stream | `DELETE /voice/api/cx/integration/v1/accounts/{rcAccountId}/sub-accounts/{subAccountId}/dialogs/{dialogId}/segments/{segmentId}/streams/{streamId}` | [Reference](https://developers.ringcentral.com/engage/voice/api-reference/Media-Control/stopSegmentStream) |

## Recommended Workflow

1. Configure the streaming profile as described in [Getting Started](getting-started.md).
2. Discover or receive the `dialogId` and, if needed, the `segmentId` for the interaction.
3. Start a dialog-level or segment-level stream.
4. Store the returned `streamId` from the start response.
5. Stop the stream with the matching `streamId` when media capture is no longer needed.

!!! warning
    Media control changes affect live audio handling. Confirm consent, retention, and compliance requirements before starting streams automatically.
