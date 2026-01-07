# Error Handling

## Overview

The vDC-API uses a structured error handling system based on generic response messages. This document describes error codes, error types, and best practices for error handling.

## Generic Response

When a command fails, the vDC returns a `GenericResponse` message containing error information.

### GenericResponse Fields

| Field | Type | Purpose |
|-------|------|---------|
| `code` | integer | Numerical result code (see Error Codes below) |
| `errorType` | integer | Kind of failure with implications for error recovery |
| `description` | string | Explanation text for developers (not for end users) |
| `userMessageToBeTranslated` | string | User-friendly message suitable for display in UIs (will be translated by dSS) |

### Field Usage

- **`errorType` and `userMessageToBeTranslated`:** Used for program flow and error recovery decisions
- **`code` and `description`:** Informational, primarily for debugging and logging
- **User Display:** Show `userMessageToBeTranslated` to users, not `description`

## Error Codes

The following error codes are defined in the Protocol Buffer schema:

| Code | Name | Number | Description |
|------|------|--------|-------------|
| `ERR_OK` | Success | 0 | Everything is alright (not an error) |
| `ERR_MESSAGE_UNKNOWN` | Unknown Message | 1 | The message ID is unknown; may indicate incomplete or incompatible implementation |
| `ERR_INCOMPATIBLE_API` | Incompatible API | 2 | The API version of the vdSM is not compatible with this vDC |
| `ERR_SERVICE_NOT_AVAILABLE` | Service Unavailable | 3 | The vDC cannot respond; might happen because the vDC is already connected to another vdSM |
| `ERR_INSUFFICIENT_STORAGE` | Insufficient Storage | 4 | The vDC could not store the related data |
| `ERR_FORBIDDEN` | Forbidden | 5 | The call is not allowed |
| `ERR_NOT_IMPLEMENTED` | Not Implemented | 6 | Feature not (yet) implemented |
| `ERR_NO_CONTENT_FOR_ARRAY` | No Array Content | 7 | Array data was expected but not provided |
| `ERR_INVALID_VALUE_TYPE` | Invalid Value Type | 8 | Invalid data type was provided |
| `ERR_MISSING_SUBMESSAGE` | Missing Submessage | 9 | A required submessage was expected but not found |
| `ERR_MISSING_DATA` | Missing Data | 10 | Additional data was expected but not provided |
| `ERR_NOT_FOUND` | Not Found | 11 | Addressed entity or object was not found |
| `ERR_NOT_AUTHORIZED` | Not Authorized | 12 | The caller is not authorized with the native device |

## Error Types

Error types provide guidance on error recovery strategies:

| Error Type | Number | Description | Recovery Strategy |
|------------|--------|-------------|-------------------|
| `ERROR_TYPE_FAILED` | 0 | Something went wrong (the usual error type) | Check error details; may or may not be retryable |
| `ERROR_TYPE_OVERLOADED` | 1 | Temporary lack of resources | **Do NOT retry immediately**; wait before retrying to avoid exacerbating the problem |
| `ERROR_TYPE_DISCONNECTED` | 2 | Communication over a lost connection | Reconnect and try again |
| `ERROR_TYPE_UNIMPLEMENTED` | 3 | Method not implemented | Revert to fallback approach using other methods |

## Error Handling Strategies

### Based on Error Type

#### ERROR_TYPE_FAILED (0)
```
If error_type == ERROR_TYPE_FAILED:
    - Log the error (code and description)
    - Examine the specific error code
    - Show userMessageToBeTranslated to user if applicable
    - Decide retry strategy based on error code
```

#### ERROR_TYPE_OVERLOADED (1)
```
If error_type == ERROR_TYPE_OVERLOADED:
    - Log the error
    - Wait with exponential backoff
    - Retry the operation after delay
    - Do NOT retry immediately
```

Example backoff:
```python
retry_delays = [1, 2, 4, 8, 16]  # seconds
for delay in retry_delays:
    time.sleep(delay)
    result = retry_operation()
    if result.code == ERR_OK:
        break
```

#### ERROR_TYPE_DISCONNECTED (2)
```
If error_type == ERROR_TYPE_DISCONNECTED:
    - Close current connection
    - Re-establish session
    - Retry the operation
```

#### ERROR_TYPE_UNIMPLEMENTED (3)
```
If error_type == ERROR_TYPE_UNIMPLEMENTED:
    - Log that feature is not available
    - Use fallback implementation
    - Do NOT retry the same method
```

### Based on Error Code

#### ERR_INCOMPATIBLE_API (2)
```
Action: Stop attempting to connect
Reason: Fundamental incompatibility
Recovery: Update vDC or vdSM to compatible version
```

#### ERR_SERVICE_NOT_AVAILABLE (3)
```
Action: Retry after delay
Reason: vDC may be connected to another vdSM
Recovery: Wait and retry, or reconfigure
```

#### ERR_NOT_IMPLEMENTED (6)
```
Action: Use alternative approach
Reason: Feature not available
Recovery: Check API version, use different method
```

#### ERR_NOT_FOUND (11)
```
Action: Verify entity exists
Reason: dSUID or resource doesn't exist
Recovery: Re-query available entities
```

#### ERR_NOT_AUTHORIZED (12)
```
Action: Authenticate or reconfigure
Reason: Missing authorization
Recovery: Provide credentials or permissions
```

## Response Handling Pattern

### For Request-Response Messages

```python
def handle_response(response):
    if response.type == GENERIC_RESPONSE:
        if response.generic_response.code != ERR_OK:
            handle_error(response.generic_response)
            return None
        # Success case (code == ERR_OK)
        return response
    else:
        # Specific response type
        return response

def handle_error(generic_response):
    error_type = generic_response.errorType
    error_code = generic_response.code
    
    # Log for developers
    log.error(f"Error {error_code}: {generic_response.description}")
    
    # User notification
    if generic_response.userMessageToBeTranslated:
        notify_user(generic_response.userMessageToBeTranslated)
    
    # Recovery strategy
    if error_type == ERROR_TYPE_OVERLOADED:
        schedule_retry_with_backoff()
    elif error_type == ERROR_TYPE_DISCONNECTED:
        reconnect_session()
    elif error_type == ERROR_TYPE_UNIMPLEMENTED:
        try_fallback_method()
```

## Best Practices

### For vDC Implementers

1. **Always include userMessageToBeTranslated** for user-facing errors
2. **Set appropriate errorType** to guide recovery
3. **Use specific error codes** to provide clear information
4. **Include helpful description** for debugging

### For vdSM Implementers

1. **Check error type first** for recovery strategy
2. **Don't retry immediately** on ERROR_TYPE_OVERLOADED
3. **Log error details** for troubleshooting
4. **Display user messages** to end users
5. **Implement exponential backoff** for retries

### General Guidelines

- **Timeout Handling:** Set reasonable timeouts for request-response operations
- **Connection Errors:** Distinguish between network errors and protocol errors
- **Logging:** Log error details but don't spam logs with expected errors
- **User Experience:** Provide actionable error messages to users
- **Testing:** Test error paths as thoroughly as success paths

## Error Scenarios

### Scenario 1: vDC Already Connected

**Situation:** vdSM tries to connect, but vDC is already connected to another vdSM

**Error Response:**
```
code: ERR_SERVICE_NOT_AVAILABLE (3)
errorType: ERROR_TYPE_FAILED (0)
description: "Already connected to another vdSM"
userMessageToBeTranslated: "Device is already in use"
```

**Recovery:** Wait and retry, or check vDC whitelist configuration

### Scenario 2: Incompatible API Version

**Situation:** vdSM API version is not supported by vDC

**Error Response:**
```
code: ERR_INCOMPATIBLE_API (2)
errorType: ERROR_TYPE_FAILED (0)
description: "API version 4 not supported (max: 3)"
userMessageToBeTranslated: "Software update required"
```

**Recovery:** Update vDC or vdSM to compatible version; cease connection attempts

### Scenario 3: Device Not Found

**Situation:** Request targets a dSUID that doesn't exist

**Error Response:**
```
code: ERR_NOT_FOUND (11)
errorType: ERROR_TYPE_FAILED (0)
description: "Device with dSUID xyz not found"
userMessageToBeTranslated: "Device not available"
```

**Recovery:** Verify dSUID; device may have been removed

### Scenario 4: Temporary Overload

**Situation:** vDC is temporarily out of resources

**Error Response:**
```
code: ERR_INSUFFICIENT_STORAGE (4)
errorType: ERROR_TYPE_OVERLOADED (1)
description: "Queue full, try again later"
userMessageToBeTranslated: "System busy, please wait"
```

**Recovery:** Wait with exponential backoff, then retry

## Debugging Tips

1. **Enable Detailed Logging:** Log all error responses with full details
2. **Use Description Field:** The description field often contains valuable debugging information
3. **Check Both Code and Type:** Error code tells what failed; error type tells how to recover
4. **Monitor Network:** Use Wireshark or tcpdump to inspect Protocol Buffer messages
5. **Test Error Cases:** Deliberately trigger errors to test error handling paths

## Next Steps

- Review [API Basics](api-basics.md) for message structure
- Learn about [Session Management](session-management.md) for connection lifecycle
- Explore [Device Operations](device-operations.md) for operation-specific errors
