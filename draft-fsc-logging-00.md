%%%
title = "FSC - delegation"
abbrev = "FSC - delegation"
ipr = "trust200902"
submissiontype = "independent"
area = "Internet"
workgroup = ""
keyword = ["Internet-Draft"]


[seriesInfo]
name = "Internet-Draft"
value = "draft-fsc-delegation-00"
stream = "independent"
status = "informational"

# date = 2022-11-01T00:00:00Z

[[author]]
initials = "E."
surname = "Hotting"
fullname = "Eelco Hotting"
organization = "Hotting IT"
  [author.address]
   email = "rfc@hotting.it"

[[author]]
initials = "R."
surname = "Koster"
fullname = "Ronald Koster"
organization = "VNG"
  [author.address]
   email = "rfc@phillyshell.nl"

[[author]]
initials = "H."
surname = "van Maanen"
fullname = "Henk van Maanen"
organization = "AceWorks"
  [author.address]
   email = "henk.van.maanen@aceworks.nl"

[[author]]
initials = "N."
surname = "Dequeker"
fullname = "Niels Dequeker"
organization = "VNG"
  [author.address]
   email = "niels.dequeker@vng.nl"

%%%

.# Abstract

TODO

{mainmatter}

# Introduction

This RFC is an extension of the Federated Service Connectivity (FSC) Core specification. This extension describes how Peers should log requests made to Services and how Peers should provide log records to other Peers.

## Purpose

Organizations should handle data in a transparent and responsible manner, partly this means that each organization should keep a log of data handled by the organization and provide insight into this log to relevant parties. FSC aims to uniform the logging of API requests to lay the groundwork for more extensive logging requirements like GDRP. 
As it is impossible to create a logging standard that will satisfy the requirements of each individual organization, FSC will focus on logging the properties of an API request of which FSC can guarantee its authenticity e.g. the Peer making the request, the Peer receiving the request, the Service that is being called etc.
Organizations can use this as a foundation to create a log that will satisfy all their needs.

## Overall Operation of Logging

A client makes a request to a Service of the Group. The Outway will receive this request and write a log record with a unique ID before proxying the request to the Inway offering the Service. The Outway will include the unique ID in the request made to the Inway. The Inway also writes a log record containing the same unique ID before proxying the request to the Service.

Peers can request log records from other Peers. Peers provide only log records in which the requesting Peer is an active party. 
 
Log records between Peers can be matched using the unique ID.

## Requirements Language {#header}

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14) [RFC2119](https://www.rfc-editor.org/rfc/rfc2119) [RFC8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals, as shown here.

## Terminology

This section lists terms (#header) and abbreviations that are used in this document. This document assumes that the reader is familiar with the Terminology of FSC Core.

*Transaction:*

A request made by a Peer to a Service.

*TransactionLog:*  

A Peers log of transactions. This log can contain both incoming and outgoing requests.

*TransactionID:*  
  
A unique identifier which can be used to trace a transaction between Peers.

# Architecture

## Writing to the TransactionLog

A Peer makes an HTTP request to a Service. The Outway will generate a unique ID for the request and write a record to the TransactionLog before proxying the request to the Inway.
The Inway will parse the unique ID from the request and also write a record containing the unique ID to its own TransactionLog before proxying the request to the Service.

!---
![Write to the TransactionLog](diagrams/seq-write-transaction-log.svg "Write to the TransactionLog")
![Write to the TransactionLog](diagrams/seq-write-transaction-log.ascii-art "Write to the TransactionLog")
!---

## Providing the TransactionLog

A Peer provides the TransactionLog to other Peers. A Peer can request the records of the TransactionLog through the Manager of a Peer. 
The Manager returns only logs records that involve the Peer requesting the log records.   

!---
![Provide the TransactionLog](diagrams/seq-provide-transaction-log.svg "Provide the TransactionLog")
![Provide the TransactionLog](diagrams/seq-provide-transaction-log.ascii-art "Provide the TransactionLog")
!---

## Connecting log records 

Each TransactionLog record will have a TransactionID which is the unique ID for the Transaction. This ID is used to link the TransactionLog records of a Transactions made across multiple Peers.
It is recommended to also add the TransactionID to logs created by other applications involved with the Transaction. E.g. the client making the request or the API offered as Service. This will enable Peers to provide a detailed audit trail of a request.   

# Specification 

## TransactionLog record{#specification_transaction_log_record}

* *TransactionID(string):*  
  A UUID[@!RFC4122] version 4 that identifies a Transaction.  
* *Direction(int32):*  
  The Direction of the request. The possible values are defined in the [Direction enum](#transaction_log_record).  
* *GrantHash(string):*  
  The hash of the Grant used to authorize the request.
* *Source(TransactionLogRecordSource):*
  Object containing information about the source of the request.    
* *Destination(TransactionLogRecordDestination):*
  Object containing information about the Destination. 
* *ServiceName(string):*
  The name of the Service being requested.  
* *CreatedAt(uint64):*
  A unix timestamp of the date on which the transaction log record was created.  

### TransactionLogRecordSource {#transaction_log_source}

* *Data(object):*  
  Object containing information about the identity of the source. Can be either [Source](#source) or [DelegatedSource](#delegated_source)  

### TransactionLogRecordDestination {#transaction_log_destination}

* *Data(object):*
  Object containing information about the identity of the destination. Can be either [Destination](#destination) or [DelegatedDestination](#delegated_destination)  

### Source{#source}

* *OutwayPeerID(string):*  
  The ID of the Peer owning the Outway making the request.

### DelegatedSource{#delegated_source}

* *OutwayPeerID(string):*  
  The ID of the Peer owning the Outway making the request.  
  *DelegatorPeerID(string):*  
  The ID of the Peer acting as Delegator.  

### Destination{#destination}

* *ServicePeerID(string):*  
  The ID of the Peer providing the Service.  

### DelegatedDestination{#delegated_destination}

* *ServicePeerID(string):*  
  The ID of the Peer providing the Service.  
* *DelegatorPeerID(string):*  
  The ID of the Peer acting as delegator.

## Manager

The FSC Log specification requires the Manager described in Core to be implemented.

### Behavior

#### Providing TransactionLog records

The Manager **MUST** be able to provide TransactionLog records to other Peers.

The Manager **MUST** only return TransactionLog records which match the following criteria:

- The Peer ID of the X.509 certificate used by the Peer requesting the TransactionLog records matches the value of the field `TransactionLogRecord.Source.OutwayPeerID`
- The Peer ID of the X.509 certificate used by the Peer requesting the TransactionLog records matches the value of the field `TransactionLogRecord.Destination.ServicePeerID`

If the Delegation extension is enabled the following two criteria also apply:

- The Peer ID of the X.509 certificate used by the Peer requesting the TransactionLog records matches the value of the field `TransactionLogRecord.Source.DelegatorPeerID`
- The Peer ID of the X.509 certificate used by the Peer requesting the TransactionLog records matches the value of the field `TransactionLogRecord.Destination.DelegatorPeerID`

### Interfaces

#### TransactionLog Record{#transaction_log_record}

```
enum Direction {
    DIRECTION_UNSPECIFIED = 0;
    DIRECTION_INCOMING = 1;
    DIRECTION_OUTGOING = 2;
}

message TransactionLogRecord {
    string transaction_id = 1;
    Direction direction = 2;
    string grant_hash = 3;
    TransactionLogRecordSource source = 4;
    TransactionLogRecordDestination destination = 5;
    string service_name = 6;
    uint64 created_at = 7;
}

message TransactionLogRecordSource {
  message Source {
    string outway_peer_id = 1;
  }

  message DelegatedSource {
    string outway_peer_id = 1;
    string delegator_peer_id = 2;
  }

  oneof data {
    Source source = 1;
    DelegatedSource delegated_source = 2;
  }
}

message TransactionLogRecordDestination {
  message Destination {
    string service_peer_id = 1;
    string service_name = 2; 
  }

  message DelegatedDestination {
    string service_peer_id = 1;
    string service_name = 2;
    string delegator_peer_id = 3;
  }

  oneof data {
    Destination destination = 1;
    DelegatedDestination delegated_destination = 2;
  }
}
```

#### RPC ListTransactionRecords

The Remote Procedure Call `ListTransactionRecords` **MUST** only return TransactionLog records involving the Peer calling the RPC.

The Remote Procedure Call `ListTransactionRecords` **MUST** be implemented with the following interface and messages:

```
enum SortOrder {
  SORT_ORDER_UNSPECIFIED = 0;
  SORT_ORDER_ASCENDING = 1;
  SORT_ORDER_DESCENDING = 2;
}

message Pagination {
  string start_id = 1;
  uint32 limit = 2; 
  SortOrder order = 3;
}

rpc ListTransactionRecords(ListTransactionLogRecordsRequest) returns (ListTransactionLogRecordsResponse);

message ListTransactionRecordsRequest{
    message Period {
        uint64 start = 1;
        uint64 end = 2;
    }
    
    message Filter {
      Period period = 1;
      string transaction_id = 2;
      string service_name = 3;
      string grant_hash = 4;
      
    }
   
    Pagination pagination = 1;
    repeated Filter filters = 2;
}

message ListTransactionRecordsResponse {
    repeated TransactionLogRecord records = 1;
}
```

## Inway

The FSC Log specification requires the Inway described in Core to be implemented.

### Behavior

#### Writing to the TransactionLog

The Inway **MUST** write a record to the TransactionLog for each received request for a Service.

The Inway **MUST** use the TransactionID provided by the Outway in the HTTP header `Fsc-Transaction-Id`.

The Inway **MUST** add the TransactionID to the request sent to the Service using the HTTP header `Fsc-Transaction-Id`.

The TransactionLog record **MUST** contain the fields described in the [TransactionLog record specification](#specification_transaction_log_record)

The Inway **MUST** deny the request if the TransactionLog record could not be created. 

#### Delegation 

When the requesting Peer is making the request on behalf of another Peer the [TransactionLogSource](#transaction_log_source) **MUST** contain a [DelegatedSource](#delegated_source) object.

When the Service is published on behalf of another Peer the [TransactionLogDestination](#transaction_log_destination) **MUST** contain a [DelegatedDestination](#delegated_destination) object.

#### Error response

This extension introduces a new error code for the Inway:

- `TRANSACTION_LOG_WRITE_ERROR`: The TransactionLog record could not be created.

## Outway

The FSC Log specification requires the Outway described in Core to be implemented.

### Behavior

#### Writing to the TransactionLog

The Outway **MUST** write a record to the TransactionLog for each request that will be sent to the Inway.

The Outway **MUST** create a TransactionID which **MUST** be unique for the transaction.

The Outway **MUST** add the TransactionID to the request sent to the Inway using the HTTP header `Fsc-Transaction-Id`.

The TransactionLog record **MUST** contain the fields described in the [TransactionLog record specification](#specification_transaction_log_record).

The Outway **MUST** deny the request if the TransactionLog record could not be written.

The Outway **MUST** add the TransactionID to the response sent to the Client using the HTTP header `Fsc-Transaction-Id`.

#### Delegation

When the requesting Peer is making the request on behalf of another Peer the [TransactionLogSource](#transaction_log_source) **MUST** contain a [DelegatedSource](#delegated_source) object.

When the Service is published on behalf of another Peer the [TransactionLogDestination](#transaction_log_destination) **MUST** contain a [DelegatedDestination](#delegated_destination) object.

#### Error response

This extension introduces a new error code for the Outway:

- `TRANSACTION_LOG_WRITE_ERROR`: The TransactionLog record could not be created.

# References

# Acknowledgements

{backmatter}
