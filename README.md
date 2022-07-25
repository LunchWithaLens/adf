## Simple guide to using Azure Data Factory (ADF) pipeline to read Project Online OData and write to a database.

In this example only the project data is being read and only a subset of columns, using $select in the Url.  
All of the records are written to a staging table.
Currently this is a one-off process with no detection of difference or incremental loads, it is really aimed at getting the authentication working. In a real world scenario you may also need to deal with a paged response - and follow new OData Urls from the 1st page.

The out of the box OData connector for ADF does not support an ROPC (Resource Owner Password Credentials) OAuth2 connection, 
and for Project Online OData this type of grant is required as just an App Id is not supported.  
It needs App Id and an identified user, so in this case ROPC is the only option.

The example hold the username, password and secrets of the App registration in Azure Key Vault.  
I also stored the Client ID and scope there, although this isn't really 'secret'.

A web activity makes the call with the required grant type set in the body.  The returned token is stored in a variable.

The token is used in the ReST activity call to the ReST linked Service and dataset, then mapped to a SQL datbase in Azure with the same limited set of fields.

4 stored procedures handle the data update from the staging table to the main table.  The first deletes the staging table data before it is re-populated.
In a live system depending on size and activity it may be better to pull just projects that have been updated recently rather than all projects as I have here in my sample.
The 2nd stored procedure deletes rows in the main table that no longer in staging, which works in my sample as I pull all current projects.  
The 3rd deletes rows in the main table that have more recent updates in the staging table, then the last stored procudure creates new records in the main table for the new and updated projects in the staging table.

In a real system you would also have tasks and assignments, and resources and custom fields - this sample was more about showing the authentication than all of the required data flows that might be needed.
