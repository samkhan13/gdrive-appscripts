# Introduction
This file provides instruction on how to use pagedFilesList.txt as a Google Apps Script.

Disclaimer: Use this example at your own risk.
License: CC0

## Main Objectives
The purpose of this script is to inspect all files within Google Drive that have been shared with someone. The results of the search include details about those who can view, comment or edit a shared file, which are tabulated in the spreadsheet associated with the script. 

In particular, you should be interested in reviewing files and folders that have been shared with:
  - ANYONE or 
  - ANYONE_WITH_LINK

## How to setup and run code.gs for pagedFilesList
The following steps will help you create and run pagedFilesList.txt as a Google Apps Script privately within your Google Drive. 

  1. After logging into your Google Drive account, create a new blank spreadsheet in a suitable location.
  2. Give this new spreadsheet a name, such as: files_permissions_audit.
  3. Add a new sheet using the plus sign at the bottom left corner of the screen so that you have exactly two blank sheets in this spreadsheet.
  4. From the menu bar click on "Tools" and then click on "Script editor" which will open the script editor in a new browser tab or window. 
  
<p align="center">
  <img width="65%" src="/img/using-script-editor-01.png"></img>
</p>
<p align="center">
Fig. 01
</p>

  5. Rename the new project to pagedFilesList
  6. On the left hand side of the code editor, click on the plus sign next to "Services" which will open a dialog box to select services that need to be added. Scroll down and select "Drive API" then click "Add".
 
<p align="center">
  <img width="65%" src="/img/using-script-editor-02.png"></img>
</p>
<p align="center">
Fig. 02
</p>
 
  7. Copy the contents of "../pagedFilesList.txt" into "Code.gs" while overwriting the sample myFunction template in it.
  8. Use Ctrl+S keyboard shortcut or the floppy icon next to "Run" in the code editor menu to save the file. 
  9. When you click on run you will be guided through a few steps to authorise this script as an "app". Click on "Review Permissions".

<p align="center">
  <img width="65%" src="/img/using-script-editor-03.png"></img>
</p>
<p align="center">
Fig. 03
</p>

  10. Next, select your signed in user account and click on "Allow" to give it access to your Google Drive files as well as run (due to automated triggers) without you being present.

<p align="center">
  <img width="49%" src="/img/using-script-editor-04.png"></img>
</p>
<p align="center">
Fig. 04
</p>

  11. You will be able to see the execution log of the running app in the script editor tab. Open https://script.google.com/home/executions in a new tab. Here you can see the particular Google Compute Platform (GCP) container running your script. Refresh the page to see the new ones. 
  12. The script will stop running after 5.5 mins and will create a trigger to run the script again in a set of 5.5 min intervals till it goes through all items in the drive. This trigger based method is needed because GCP prevents a script from continuously running for longer than 6 min. Each trigger gets "Disabled" after being launched once.
  13. In the tab https://script.google.com/home/triggers you will be able to see those automated triggers being created every 5.5 min. Refresh the page to see the new ones.
  14. In the spreadsheet you will be able to see the data get populated in "Sheet1".
  15. In "Sheet2" you will see a few pieces of data that are used for tracking the run number, the pageToken and itemCount between runs. Don't modify these while the script is running. 
  16. Delete all cells of "Sheet1" and "Sheet2" if the runs fail and you want to start over. 
  17. You can change the query and start with a new spreadsheet if you like. The spreadsheet needs to have a blank "Sheet1" and "Sheet2" at the beginning of running the script. 

## Changing Global Variables
  - `maxExecTime` is a positive number that can be changed as needed. It needs to be in milliseconds, sufficiently large for the first run to complete and less than 360000 milli seconds.
  - `dwellTime` is a positive number in milliseconds. It is the time period between when the script stops and is triggered again. It can be changed as needed.

  - `maxQueryItems` is a positive number less than 1000. It is 100 by default for all queries via `Drive.Files.list()`. I prefered using a smaller number and while experimentin with it I noticed that it never went above 460 for higher values.
  - `query` is a string for serching for various file types. More on this in the following section. 

### Acceptable Query Format
These examples can hopefully provide insights into how a query string can be put together.
  - `"'me' in owners"` 
    - This will search for all files within the scope of "My Drive", which is the root directory of Google Drive. 


  - `"'me' in owners and trashed = false"`
    - This will exclude all files that are currently in the "Trash" folder. 


  - `"mimeType contains 'audio'"`
    - This will find all files, including ones shared with you, that are any type of audio file.
    - Examples of `mimeType` for audio files are: `'application/vnd.google-apps.audio'`, `'audio/mpeg'`, `'audio/mp3'`, etc.


  - `"not 'me' in owners and trashed = true and mimeType='video/mp4' and title contains 'new'"`
    - This searches only in the drive's "Trash" folder for shared video files that have the sub-string "new" in their name.
    - The current documentation indicates that the query_term `name` can be used however that did not work and only `title` works. This could be because `Drive.Files.List()` returns an `items` object and an item within `items` has the property of `item.title`. More about this is discussed in the Notes section.


  - `"'me' in owners and not mimeType contains 'image' and mimeType='applicatoin/jason' and mimeType contains 'text'"`
    - Finds everything in "My Drive" that is owned and is a jason file as well of the type: `text/plain` and `text/xml`.  
    - Other interesting file types can be `'applicaton/x-javascript'`, `application/octet-stream`, `application/x-shellscript`, `application/pdf`, etc.


  - `"'me' in owners and mimeType='application/vnd.google-apps.folder'"`
    - Returns all folders including nested subfolders. 
    - There is actually no such thing as a "folder" in Google Drive and everything is a file within a Google Cloud Storage bucket (Google's simple storage solution).
    - It isn't necessary to find a "folder" and then "recursively" go through files or sub-folders within it. 
    - Faster results are obtained by using a better structured query string.  

  - `"title contains 'office' and starred=false"`
    - Finds all files and folders with the sub-string office in their title (i.e. name) and that don't happen be starred in "My Drive".

## Notes
1. Only Drive API V2 allows `pageToken` based queries using `Drive.Files.List()`. This is needed to overcome "Time Out" issues of running Apps Script via default Google Compute Platform. However, a number of things like "sharingAccess" and "sharingPermission" are more conviniently usable via Drive App API. Using both API means that the same file's details are obtained twice using a slow "GET" operation. 
2. An array of [File "object"](https://developers.google.com/drive/api/v3/reference/files) of `"kind": "drive#file"`, as described in Drive API V3, is fethced via `Drive.Files.List()` even though `Drive.Files.List()` with `pageToken` is only described in Drive API V2 "[Advanced Google Services Examples](https://developers.google.com/apps-script/advanced/drive#listing_folders)". 
3. Also, `pageSize` as a keyword doesn't work in `Drive.Files.List()`, the key used is `maxResults`.
4. The above things are considerably different than a [File "class"](https://developers.google.com/apps-script/reference/drive/file) fetched via DriveApp API. 
5. It is possible that legacy API was handed over to a new team that decided to update it by "starting from scratch." So now the different versions are very inconsistent compared to eachother. 
6. The words used in Google Drive for file permissions like "View", "Comment" or "Edit" are more in line with DriveApp V2 API. However, in Drive API these permissions are identified as "reader" and "writer" which doesn't translate properly for answering, "who can write only comments vs write content as well as comments for a file?"
7. There are lots of wierd ways in which a file's parent folder might have different permissions that inherited but were later modified manually. There are many more wierd ways in which a file's parent folder may not exist anymore in any Google Storage bucket. 
8. Drive API is designed to get only some default descriptors of a file. So not all fields of a file are fetched and more effort needs to be put into reading documentation for ".
9. According to: https://developers.google.com/drive/api/v3/about-files#file_organization
    - An item in Drive is a file.    
    - Folders are files with a metadata i.e `mimeType = 'application/vnd.google-apps.folder'`
    - An item's permisions can be changed if 
      - it is owned by a user, 
      - a user was granted "writer" permission for the item with "writerCanShare = true"
10. According to: https://developers.google.com/drive/api/v3/search-files
    - `Drive.Files.list()` without any parameters returns all items in user's "My Drive". But this will time-out after six minutes without using pageTokens and triggers.
11. There is most probably a simple way to spool rows that need to be written into the sheet and only send one "POST" method ever so often instead of calling the method per row. This will speed up things and make the code more efficient. 
12. Please do fork and improve this. Please send pull request with better refactored code. 


## References:
- https://developers.google.com/drive/api/v3/search-files
- https://developers.google.com/drive/api/v3/ref-search-terms
- https://developers.google.com/drive/api/v3/mime-types

- Currently available references for Drive API using Apps Script: 
  - https://developers.google.com/apps-script/reference/drive/
  - https://developers.google.com/apps-script/advanced/drive 

- DriveApp based sharingAccess and sharingPermission reference:
  - https://developers.google.com/apps-script/reference/drive/access
  - https://developers.google.com/apps-script/reference/drive/permission

## Inspirations
-  [woodwardtw/tellmeyoursecrets.js](https://gist.github.com/woodwardtw/22a199ecca73ff15a0eb) 
-  [ichaer/tellmeyoursecrets.js](https://gist.github.com/ichaer/d7b91d348a250e09146057857f7b3cc2)
-  [rzrbld/tellmeyoursecrets.js](https://gist.github.com/rzrbld/ba50a0f51b081a5699cb1d4996e4925a)
-  [danjargold/whatFilesHaveIShared.gs](https://gist.github.com/danjargold/c6542e68fe3a3b46eeb0172f914641bc)
-  [JavierCane/listGoogleDriveSharedDocuments.js](https://gist.github.com/JavierCane/28f7307ceeaf6464431c1418b598a817)
-  [moya-a/G-Drive-SharedFiles-Checker/checker.js](https://github.com/moya-a/G-Drive-SharedFiles-Checker/blob/main/checker.js)

Thank you. 
