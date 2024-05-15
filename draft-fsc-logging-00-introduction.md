# Introduction

The Logging specification is an extension of the Federated Service Connectivity (FSC) Core specification. This extension describes how Peers should log requests made to Services and how Peers should provide log records to other Peers.

## Purpose

Organizations should handle data in a transparent and responsible manner, partly this means that each organization should keep a log of data handled by the organization and provide insight into this log to relevant parties. FSC aims to uniform the logging of API requests to lay the groundwork for more extensive logging requirements like GDRP.
As it is impossible to create a logging standard that will satisfy the requirements of each individual organization, FSC will focus on logging the properties of an API request of which FSC can guarantee its authenticity e.g. the Peer making the request, the Peer receiving the request, the Service that is being called etc.
Organizations can use this as a foundation to create a log that will satisfy all their needs.

## Overall Operation of Logging

A client makes a request to a Service of the Group. The Outway will receive this request and write a log record with a unique ID before proxying the request to the Inway offering the Service. The Outway will include the unique ID in the request made to the Inway. The Inway also writes a log record containing the same unique ID before proxying the request to the Service.

Peers can request log records from other Peers. Peers provide only log records in which the requesting Peer is an active party.

Log records between Peers can be matched using the unique ID.

## Terminology

This section lists terms (#header) and abbreviations that are used in this document. This document assumes that the reader is familiar with the Terminology of FSC Core.

*Transaction:*

A request made by a Peer to a Service.

*TransactionLog:*

A Peers log of Transactions. This log can contain both incoming and outgoing requests.

*TransactionID:*

A unique identifier which can be used to trace a Transaction between Peers.

## Profiles
When using the Logging Extension the following additions **MUST** be made to the [FSC Profile](../core/draft-fsc-core-00.html#profiles):
1. Determine the expiration date for log records

In addition, the mandatory decisions a Profile **MAY** also contain additional agreements or restrictions within the Group. These are not technically required for the operation of FSC Logging extension, but can become mandatory within a Group. For example an additional set of rules to comply with local legislation.
Below are a few examples listed of these additional decisions for inspirational purposes:
1. formatting restrictions on the TransactionID, for example UUIDv7
