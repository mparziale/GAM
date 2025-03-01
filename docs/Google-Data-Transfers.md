# Google Data Transfers
- [API documentation](#api-documentation)
- [Definitions](#definitions)
- [Display transfer apps](#display-transfer-apps)
- [Create transfers](#create-transfers)
- [Display transfers](#display-transfers)

## API documentation
* https://developers.google.com/admin-sdk/data-transfer/v1/reference/transfers
* https://support.google.com/a/answer/1247799
    * Explains how background drive transfers work, including orphaned files, trashed file behaviour (not transfered), etc.

## Definitions
```
<DataTransferService> ::=
        calendar|
        currents|
        datastudio|"google data studio"|
        drive|gdrive|googledrive|"drive and docs"
<DataTransferServiceList> ::= "<DataTransferService>(,<DataTransferService>)*"

<UniqueID> ::= id:<String>
<UserItem> ::= <EmailAddress>|<UniqueID>|<String>
<OldOwnerID> ::= <UserItem>
<NewOwnerID> ::= <UserItem>
<TransferID> ::= <String>
```

## Display transfer apps
```
gam print|show transferapps
```

## Create transfers
```
gam create|add datatransfer|transfer <OldOwnerID> <DataTransferServiceList> <NewOwnerID>
        [private|shared|all] [privacy_level private|shared|private,shared]
        [releaseresources [<Boolean>]]
        (<ParameterKey> <ParameterValue>)*
```
For`datastudio` and `drive`, there are options to control the privacy level of the files to be transferred.
* `private` or `privacy_level private` - Transfer files that are not shared with anyone
* `shared` or `privacy_level shared` - Transfer files shared with at least one other user; this is the **default**
* `all` or `privacy_level private,shared` - Transfer all files

For calendars, there is an option to indicate whether to release resources for future events.
* `releaseresources false` - Do not release resources for future events; this is the default.
* `releaseresources` or `releaseresources true` - Release resources for future events

A `<TransferID>` is returned which can be used to monitor the progress of the transfer.

NOTE: For calendars, the behaviour is not sufficiently defined in the API documentation.
As of 2020-06-10, background transfers only transfer future non-private events with at least one guest/resource.

The option `<ParameterKey> <ParameterValue>` is for future expansion.

## Display transfers
```
gam info datatransfer|transfer <TransferID>
gam show datatransfers|transfers
        [olduser|oldowner <UserItem>] [newuser|newowner <UserItem>]
        [status completed|failed|inprogress|<String>] [delimiter <Character>]
gam print datatransfers|transfers [todrive <ToDriveAttribute>*]
        [olduser|oldowner <UserItem>] [newuser|newowner <UserItem>]
        [status completed|failed|inprogress|<String>] [delimiter <Character>]
```
By default, all data transfer operations are printed, use these options to select specific transfers.
* `olduser|oldowner <UserItem>`
* `newuser|newowner <UserItem>`
* `status completed|failed|inprogress`

By default, the entries in lists of items are separated by the `csv_output_field_delimiter` from `gam.cfg`.
* `delimiter <Character>` - Separate list items with `<Character>`

