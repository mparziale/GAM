# Users - Drive - Activity/Settings
- [API documentation](#api-documentation)
- [Query documentation](Users-Drive-Query)
- [Definitions](#definitions)
- [Display drive activity](#display-drive-activity)
- [Display drive settings](#display-drive-settings)

## API documentation
* https://developers.google.com/drive/api/v3/reference/files
* https://developers.google.com/drive/activity/v2/migrating
* https://developers.google.com/drive/activity/v2/reference/rest

## Definitions
* [`<UserTypeEntity>`](Collections-of-Users)

```
<Date> ::=
        <Year>-<Month>-<Day> |
        (+|-)<Number>(d|w|y) |
        never|
        today
<Time> ::=
        <Year>-<Month>-<Day>(<Space>|T)<Hour>:<Minute>:<Second>[.<MilliSeconds>](Z|(+|-(<Hour>:<Minute>))) |
        (+|-)<Number>(m|h|d|w|y) |
        never|
        now|today

<DriveFileID> ::= <String>
<DriveFileName> ::= <String>
<DriveFolderID> ::= <String>
<DriveFolderName> ::= <String>

<DriveActivityAction> ::=
        comment|
        create|
        delete|trash|
        dlpchange|
        edit|
        emptytrash|
        move|
        permissionchange|
        reference|
        rename|
        restore|untrash|
        settingschange|
        upload
<DriveActivityActionList> ::= "<DriveActivityAction>(,<DriveActivityAction>)*"

<DriveSettingsFieldName> ::=
        appinstalled|
        drivethemes|
        exportformats|
        foldercolorpalette|
        importformats|
        largestchangeid|
        limit|
        maximportsizes|
        maxuploadsize|
        name|
        permissionid|
        rootfolderid|
        usage|
        usageindrive|
        usageindrivetrash
<DriveSettingsFieldNameList> ::= "<DriveSettingsFieldName>(,<DriveSettingsFieldName>)*"
```

## Display drive activity
Google has introduced Drive Activity API v2; it adds time and action filtering and has reorganized the format of the data.
Drive Activity API v1 has been deprecated.
* https://developers.google.com/drive/activity/v2/migrating
```
gam <UserTypeEntity> print|show driveactivity [v2] [todrive <ToDriveAttributes>*]
        [(fileid <DriveFileID>)|(folderid <DriveFolderID>)|
         (drivefilename <DriveFileName>)|(drivefoldername <DriveFolderName>)|
         (query <QueryDriveFile>)]
        [([start|starttime <Date>|<Time>] [end|endtime <Date>|<Time>])|(range <Date>|<Time> <Date)|<Time>|
         yesterday|today|thismonth|(previousmonths <Integer>)]
        [action|actions [not] <DriveActivityActionList>]
        [consolidationstrategy legacy|none]
        [idmapfile <FileName>|(gsheet <UserGoogleSheet>) [charset <String>] [columndelimiter <Character>] [quotechar <Character>]]
        [formatjson [quotechar <Character>]]
```
By default, Drive Activity API v2 is used; the `v2` option is ignored.

By default, drive activity for all files in the top level of My Drive will be displayed.
* `fileid <DriveFileID>` - Display drive activity for file `<DriveFileID>`
* `folderid <DriveFolderID>` - Display drive activity for all files in folder `<DriveFolderID>`
* `drivefilename <DriveFileName>` - Display drive activity for the file with name `<DriveFolderID>`
* `drivefoldername <DriveFolderName>` - Display drive activity for all files in the folder  with name `<DriveFolderName>`
* `query`  - Display drive activity for all files/folders selected by the query

Activities can be filtered by time.
* `start|starttime <Date>|<Time>` - Display activities that occurred on or after the specifed `<Date>|<Time>`
* `end|endtime <Date>|<Time>` - Display activities that occurred on or before the specifed `<Date>|<Time>`
* `range <Date>|<Time> <Date>|<Time>` - Equivalent to `start <Date>|<Time> end <Date>|<Time>`
* `yesterday` - Display activities that occurred yesterday
* `today` - Display activities that occurred today
* `thismonth` - Display activities that occurred in the current calendar month up to the current time
* `previousmonths <Integer>` - Display activities that occurred in the 1 to 6  months previous to the current month

Activities can be filtered by action.
Google does the filtering.
* `action|actions <DriveActivityActionList>` - Only display activities with the specified actions; by default, all actions are displayed
* `action|actions not <DriveActivityActionList>` - Only display activities without the specified actions; by default, all actions are displayed

The API only returns a permissionId and user name for each event but no user email address. To get an email address perform the
following command to generate a file that contains the email address and permissionId for all users. You can substitute for `all users` if desired.
```
gam config auto_batch_min 1 redirect csv DriveSettings.csv multiprocess all users print drive settings fields permissionid
```
Then, use the `idmapfile DriveSettings.csv` option to have Gam generate a `user.emailAddress` column. The columns `email' and `permissionId`
must be present in the file.

In the v2 API, the style of response does not change based on the consolidation strategy, as the returned DriveActivity always contains the full set of actors, targets, and actions.

By default, Gam calls the API with a consolidation strategy of `none`; use these options to alter that behavior
* `consolidationstrategy legacy` - The individual activities are consolidated using the legacy strategy
* `consolidationstrategy none` - The individual activities are not consolidated; this is the default

The API only returns a userId for each event but no user name or  email address. To get a name and email address perform the
following command to generate a file that contains the email address and permissionId for all users. You can substitute for `all users` if desired.
```
gam redirect csv ./Users.csv print users fields primaryemail,id,name
```
Then, use the `idmapfile Users.csv` option to have Gam generate a `user.emailAddress` column. The columns `primaryEmail' and `id`
must be present in the file; the column `name.fullName` will be used if present.

If you don't use the `idmapfile` option, Gam makes an additional API call per user to get the name and email address.

By default, Gam displays the information as columns of fields; the following option causes the output to be in JSON format,
* `formatjson` - Display the fields in JSON format.

By default, when writing CSV files, Gam uses a quote character of double quote `"`. The quote character is used to enclose columns that contain
the quote character itself, the column delimiter (comma by default) and new-line characters. Any quote characters within the column are doubled.
When using the `formatjson` option, double quotes are used extensively in the data resulting in hard to read/process output.
The `quotechar <Character>` option allows you to choose an alternate quote character, single quote for instance, that makes for readable/processable output.
`quotechar` defaults to `gam.cfg/csv_output_quote_char`. When uploading CSV files to Google, double quote `"` should be used.

## Display drive settings
```
gam <UserTypeEntity> print drivesettings [todrive <ToDriveAttribute>*]
        [allfields|<DriveSettingsFieldName>*|(fields <DriveSettingsFieldNameList>)]
        [delimiter <Character>]
gam <UserTypeEntity> show drivesettings
        [allfields|<DriveSettingsFieldName>*|(fields <DriveSettingsFieldNameList>)]
        [delimiter <Character>]
```
If no fields are selected, these fields will be displayed:
    `name,appInstalled,largestChangeId,limit,maxUploadSize,permissionId,rootFolderId,usage,usageInDrive,usageInDriveTrash`
