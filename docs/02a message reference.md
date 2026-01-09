## Message Type Reference

### Session Management

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_REQUEST_HELLO | vdsm_RequestHello | vdSM → vDC | vdc_ResponseHello |
| VDC_RESPONSE_HELLO | vdc_ResponseHello | vDC → vdSM | N/A |
| VDSM_SEND_BYE | vdsm_SendBye | vdSM → vDC | None |

### Property Access

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_REQUEST_GET_PROPERTY | vdsm_RequestGetProperty | vdSM → vDC | vdc_ResponseGetProperty |
| VDC_RESPONSE_GET_PROPERTY | vdc_ResponseGetProperty | vDC → vdSM | N/A |
| VDSM_REQUEST_SET_PROPERTY | vdsm_RequestSetProperty | vdSM → vDC | GenericResponse |
| VDC_SEND_PUSH_PROPERTY | vdc_SendPushProperty | vDC → vdSM | Error only |
| VDC_SEND_PUSH_NOTIFICATION | vdc_SendPushNotification | vDC → vdSM | Error only |

### Device Lifecycle

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDC_SEND_ANNOUNCE_VDC | vdc_SendAnnounceVdc | vDC → vdSM | Error only |
| VDC_SEND_ANNOUNCE_DEVICE | vdc_SendAnnounceDevice | vDC → vdSM | Error only |
| VDC_SEND_VANISH | vdc_SendVanish | vDC → vdSM | Error only |
| VDSM_SEND_REMOVE | vdsm_SendRemove | vdSM → vDC | None |
| VDC_SEND_IDENTIFY | vdc_SendIdentify | vDC → vdSM | Error only |

### Keep-Alive

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_SEND_PING | vdsm_SendPing | vdSM → vDC | vdc_SendPong |
| VDC_SEND_PONG | vdc_SendPong | vDC → vdSM | N/A |

### Notifications (Scene Operations)

All notifications: vdSM → vDC, no response expected

| Type | Message | Purpose |
|------|---------|---------|
| VDSM_NOTIFICATION_CALL_SCENE | vdsm_NotificationCallScene | Activate scene |
| VDSM_NOTIFICATION_SAVE_SCENE | vdsm_NotificationSaveScene | Save current state as scene |
| VDSM_NOTIFICATION_UNDO_SCENE | vdsm_NotificationUndoScene | Undo scene |
| VDSM_NOTIFICATION_SET_LOCAL_PRIO | vdsm_NotificationSetLocalPrio | Set local priority |
| VDSM_NOTIFICATION_CALL_MIN_SCENE | vdsm_NotificationCallMinScene | Call minimum scene |
| VDSM_NOTIFICATION_IDENTIFY | vdsm_NotificationIdentify | Identify device(s) |
| VDSM_NOTIFICATION_SET_CONTROL_VALUE | vdsm_NotificationSetControlValue | Set control value |
| VDSM_NOTIFICATION_DIM_CHANNEL | vdsm_NotificationDimChannel | Dim channel |
| VDSM_NOTIFICATION_SET_OUTPUT_CHANNEL_VALUE | vdsm_NotificationSetOutputChannelValue | Set channel value |

### Generic Request (API v2c+)

| Type | Message | Direction | Response |
|------|---------|-----------|----------|
| VDSM_REQUEST_GENERIC_REQUEST | vdsm_RequestGenericRequest | vdSM → vDC | GenericResponse |

