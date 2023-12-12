# Architecture

## Writing to the TransactionLog

A Peer makes an HTTP request to a Service. The Outway will generate a unique ID for the Transaction and write a record to the TransactionLog before proxying the request to the Inway.
The Inway will parse the unique ID from the request and also write a record containing the unique ID to its own TransactionLog before proxying the request to the Service.

> The storage of the log record MAY be implemented both synchronously or asynchronously. For both implementations it is REQUIRED to receive confirmation that the log record is persisted in order to continue.
> For example, you can introduce a message broker to improve performance. The message broker will ensure the records are persisted later on.

![Write to the TransactionLog](diagrams/seq-write-transaction-log.svg "Write to the TransactionLog")

## Providing the TransactionLog

A Peer provides the TransactionLog to other Peers. A Peer can request the records of the TransactionLog through the Manager of a Peer.
The Manager returns only logs records that involve the Peer requesting the log records.

![Provide the TransactionLog](diagrams/seq-provide-transaction-log.svg "Provide the TransactionLog")

## Connecting log records

Each log record will have a TransactionID which is the unique ID for the Transaction. This ID is used to link the log records of a Transaction made across multiple Peers.
It is **RECOMMENDED** to also add the TransactionID to logs created by other applications involved with the Transaction. E.g. the client making the request or the API offered as Service. This will enable Peers to provide a detailed audit trail of a request.   
