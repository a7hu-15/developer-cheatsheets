# 🔌 gRPC & Protocol Buffers (Protobuf) Architecture Reference Cheatsheet

A developer reference guide for Protocol Buffers (`proto3`) syntax, gRPC RPC communication patterns, status codes, deadlines/timeouts, metadata passing, interceptors, and high-performance RPC architecture.

---

## 📜 1. Protocol Buffers (`proto3`) Syntax

### Basic Message & Scalar Types
```protobuf
syntax = "proto3";

package user.v1;

option go_package = "github.com/example/user/v1;userv1";

enum UserStatus {
  USER_STATUS_UNSPECIFIED = 0; // Default zero-value
  USER_STATUS_ACTIVE = 1;
  USER_STATUS_SUSPENDED = 2;
}

message UserProfile {
  uint64 id = 1;               # Field tag number (1-15 use 1 byte)
  string email = 2;
  string display_name = 3;
  UserStatus status = 4;
  repeated string tags = 5;    # List / Array type
  map<string, string> metadata = 6;
}
```

### Advanced Schema Constructs
```protobuf
import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";

message GetUserRequest {
  oneof identifier {
    uint64 user_id = 1;
    string email = 2;
  }
}

message UserEvent {
  string event_id = 1;
  google.protobuf.Timestamp timestamp = 2;
  UserProfile profile = 3;
}
```

---

## ⚡ 2. gRPC Service Communication Patterns

### Service Definition
```protobuf
service UserService {
  // 1. Unary RPC (Request-Response)
  rpc GetUser(GetUserRequest) returns (UserProfile);

  // 2. Server Streaming RPC (Single Request -> Stream of Responses)
  rpc ListUsers(ListUsersRequest) returns (stream UserProfile);

  // 3. Client Streaming RPC (Stream of Requests -> Single Response)
  rpc UploadBatchUsers(stream UserProfile) returns (BatchUploadSummary);

  // 4. Bi-directional Streaming RPC (Stream of Requests <-> Stream of Responses)
  rpc LiveUserChat(stream ChatMessage) returns (stream ChatMessage);
}
```

---

## 🛑 3. gRPC Status Codes & Error Handling

| Code | Name | Description | HTTP Equivalent |
|---|---|---|---|
| `0` | `OK` | Success | 200 OK |
| `1` | `CANCELLED` | Operation cancelled by caller | 499 Client Closed |
| `3` | `INVALID_ARGUMENT` | Client specified an invalid argument | 400 Bad Request |
| `4` | `DEADLINE_EXCEEDED` | Deadline expired before operation finished | 540 Gateway Timeout |
| `5` | `NOT_FOUND` | Requested entity was not found | 404 Not Found |
| `6` | `ALREADY_EXISTS` | Entity already exists | 409 Conflict |
| `7` | `PERMISSION_DENIED` | Caller does not have required permissions | 403 Forbidden |
| `16` | `UNAUTHENTICATED` | Request does not have valid authentication | 401 Unauthorized |

---

## ⏱️ 4. Deadlines, Metadata & Interceptors

### Deadlines & Context Timeouts
```python
# Python gRPC Client with Timeout
import grpc
import user_pb2
import user_pb2_grpc

channel = grpc.insecure_channel('localhost:50051')
stub = user_pb2_grpc.UserServiceStub(channel)

try:
    response = stub.GetUser(
        user_pb2.GetUserRequest(user_id=123),
        timeout=2.5 # Timeout in seconds
    )
except grpc.RpcError as e:
    if e.code() == grpc.StatusCode.DEADLINE_EXCEEDED:
        print("RPC timed out after 2.5s!")
```

### Metadata (Headers & Binary Metadata)
```python
# Passing Custom Metadata Headers
metadata = (
    ('authorization', 'Bearer eyJhbGciOi...'),
    ('x-request-id', 'req-987654321'),
)
response = stub.GetUser(request, metadata=metadata)
```

---

## 🚀 5. Performance Best Practices

1. **Reuse Channels**: gRPC channels maintain persistent HTTP/2 connections; creating new channels per RPC causes severe overhead.
2. **Keep Field Tags <= 15**: Field tag numbers 1 through 15 take only 1 byte in the binary wire format.
3. **Use Protocol Buffers over JSON**: Binary serialization is up to 5-10x faster with significantly smaller payload size.
4. **Configure HTTP/2 Keepalives**: Prevent idle connection termination by cloud load balancers.
5. **Use Connection Pooling for Heavy Load**: Scale client throughput by multiplexing across multiple sub-channels when max HTTP/2 streams per connection are reached.
