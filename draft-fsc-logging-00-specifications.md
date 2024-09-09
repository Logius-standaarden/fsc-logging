# Specification

## Log record {#log_record}

The fields that a log record **MUST** contain are described in the [OpenAPI Specification](logging.yaml)

### Access token

Data from the access token **MUST** be used to fill the following fields of the log record:

`accessToken.gth` -->  `logRecord.grant_hash`
`accessToken.sub` -->  `logRecord.source.outway_peer_id`
`accessToken.iss` -->  `logRecord.destination.service_peer_id`
`accessToken.svc` -->  `logRecord.service_name`

in case of a Peer making a request on behalf of another Peer an additional field **MUST** be set:

`accessToken.cdi` --> `logRecord.source.delegator_peer_id`

in case of a request made to a Service offered on behalf of another Peer and additional field **MUST** be set:

`accessToken.pdi` --> `logRecord.destination.delegator_peer_id`

## Manager

The FSC Log specification requires the Manager described in Core to be implemented.

### Behavior

#### Providing TransactionLog records

The Manager **MUST** be able to provide log records to other Peers.

The Manager **MUST** only return log records which match any of the following criteria:

- The Peer ID of the X.509 certificate used by the Peer requesting the log records matches the value of the field `logRecord.source.outway_peer_id`
- The Peer ID of the X.509 certificate used by the Peer requesting the log records matches the value of the field `logRecord.destination.service_peer_id`
- The Peer ID of the X.509 certificate used by the Peer requesting the log records matches the value of the field `logRecord.source.delegator_peer_id`
- The Peer ID of the X.509 certificate used by the Peer requesting the TransactionLog records matches the value of the field `logRecord.destination.delegator_peer_id`

### Interface

The Manager **MUST** implement the interface described in the [OpenAPI Specification](logging.yaml)

## Inway

The FSC Log specification requires the Inway described in Core to be implemented.

### Behavior

#### Writing to the TransactionLog

The Inway **MUST** write a record to the TransactionLog for each received request for a Service.

The Inway **MUST** use the TransactionID provided by the Outway in the HTTP header `Fsc-Transaction-Id`.

The Inway **MUST** add the TransactionID to the request sent to the Service using the HTTP header `Fsc-Transaction-Id`.

The TransactionLog record **MUST** contain the fields described in the [log record section](#log_record)

The Inway **MUST** deny the request if the record to the TransactionLog could not be written.

#### Delegation

When the requesting Peer is making the request on behalf of another Peer the source of a log record **MUST** contain a sourceDelegated object as described in the [OpenAPI Specification](logging.yaml).

When the Service is published on behalf of another Peer the destination of a log record **MUST** contain a destinationDelegated as described in the [OpenAPI Specification](logging.yaml).

#### Error response

This extension introduces a new error code for the Inway:

| Error code                  | HTTP status code | Description                                                |
|-----------------------------|------------------|------------------------------------------------------------|
| TRANSACTION_LOG_WRITE_ERROR | 500              | The TransactionLog record could not be created             |
| INVALID_LOG_RECORD_ID       | 400              | The format of the `Fsc-Transaction-Id` header is not valid |
| MISSING_LOG_RECORD_ID       | 400              | The the `Fsc-Transaction-Id` header is missing             |

## Outway

The FSC Log specification requires the Outway described in Core to be implemented.

### Behavior

#### Writing to the TransactionLog

The Outway **MUST** write a record to the TransactionLog for each request that will be sent to the Inway.

The Outway **MUST** create a TransactionID which **MUST** be unique for the transaction, the format is determined in a FSC Profile.

The Outway **MUST** add the TransactionID to the request sent to the Inway using the HTTP header `Fsc-Transaction-Id`.

The TransactionLog record **MUST** contain the fields described in the [TransactionLog record section](#transaction_log_record)

The Outway **MUST** deny the request if the record to the TransactionLog could not be written.

The Outway **MUST** add the TransactionID to the response sent to the Client using the HTTP header `Fsc-Transaction-Id`.

#### Delegation

When the requesting Peer is making the request on behalf of another Peer the source of a log record **MUST** contain a sourceDelegated object as described in the [OpenAPI Specification](logging.yaml).

When the Service is published on behalf of another Peer the destination of a log record **MUST** contain a destinationDelegated as described in the [OpenAPI Specification](logging.yaml).

#### Error response

This extension introduces a new error code for the Outway:

| Error code                                          | HTTP status code | Description                                                |
|-----------------------------------------------------|------------------|------------------------------------------------------------|
| TRANSACTION_LOG_WRITE_ERROR                       |  500             | The TransactionLog record could not be created             |
| INVALID_LOG_RECORD_ID       | 400              | The format of the `Fsc-Transaction-Id` header is not valid |
| MISSING_LOG_RECORD_ID       | 400              | The the `Fsc-Transaction-Id` header is missing             |
