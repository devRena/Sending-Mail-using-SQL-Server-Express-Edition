# Sending Mail using SQL Server Express Edition

In SQL server standard and enterprise edition,A DataBase mail functionality in built to sent mail.
But in SQL Server express edition,You need either use CLR integration or configure SQL Mail using 
MSDB system database.
Here I am discussing sending mail using MSDB system database.By default the MSDB database installed 
when you install SQL Server.

## Database Mail Setup:

To configure SQL mail we need to follow below steps.

* Create Database Mail Account
* Creating Database Profile
* Add database Mail account to profile
* Grants permission for a database user or role to use a Database Mail profile.

### Setting up Accounts & Profiles for DB Mail:

``` --// Create a Database Mail account
	EXECUTE msdb.dbo.sysmail_add_account_sp
		@account_name = 'Test Mail Account',
		@description = 'Mail account for administrative e-mail.',
		@email_address = 'abc@xyz.com',
		@replyto_address = 'abc@xyz.com',
		@display_name = 'Manoj Pandey',
		@mailserver_name = 'smtp.xxxx.net',
		@port = 587,
		@username = 'xyz',
		@password = 'xxyyzz',
		@enable_ssl = 1
	 
	-- Create a Database Mail profile
	EXECUTE msdb.dbo.sysmail_add_profile_sp
		@profile_name = 'Test Mail Profile',
		@description = 'Profile used for administrative mail.'
	 
	-- Add the account to the profile
	EXECUTE msdb.dbo.sysmail_add_profileaccount_sp
		@profile_name = 'Test Mail Profile',
		@account_name = 'Test Mail Account',
		@sequence_number =1
	 
	-- Grant access to the profile to the DBMailUsers role
	EXECUTE msdb.dbo.sysmail_add_principalprofile_sp
		@profile_name = 'Test Mail Profile',
		@principal_name = 'public',
		@is_default = 1  
```
	
### Execute the following Stored-Porcs

``` exec dbo.sysmail_start_sp
```
```
exec dbo.sysmail_stop_sp 
```
	
### To Enable Database Mail execute the following block of code:
``` use master
	go
	exec sp_configure 'show advanced options', 1
	reconfigure
	exec sp_configure 'Database Mail XPs', 1
	reconfigure 
```
	
## Important Tables used in configuring Database mail and check their status:
```  --Profiles
	SELECT * FROM msdb.dbo.sysmail_profile
 
	--Accounts
	SELECT * FROM msdb.dbo.sysmail_account
 
	--Profile Accounts
	select * from msdb.dbo.sysmail_profileaccount
 
	--Principal Profile
	select * from msdb.dbo.sysmail_principalprofile
	 
	--Mail Server
	SELECT * FROM msdb.dbo.sysmail_server
	SELECT * FROM msdb.dbo.sysmail_servertype
	SELECT * FROM msdb.dbo.sysmail_configuration
	 
	--Email Sent Status
	SELECT * FROM msdb.dbo.sysmail_allitems
	SELECT * FROM msdb.dbo.sysmail_sentitems
	SELECT * FROM msdb.dbo.sysmail_unsentitems
	SELECT * FROM msdb.dbo.sysmail_faileditems
	 
	--Email Status
	SELECT SUBSTRING(fail.subject,1,25) AS 'Subject',
		   fail.mailitem_id,
		   LOG.description
	FROM msdb.dbo.sysmail_event_log LOG
	join msdb.dbo.sysmail_faileditems fail
	ON fail.mailitem_id = LOG.mailitem_id
	WHERE event_type = 'error'
	 
	--Mail Queues
	EXEC msdb.dbo.sysmail_help_queue_sp
	 
	--DB Mail Status
	EXEC msdb.dbo.sysmail_help_status_sp 
```
	
	
## Ready to send an email

```  --Send mail
	EXEC msdb.dbo.sp_send_dbmail
	@recipients='abc@xyz.gr',
	@body= 'Test Email Body',
	@subject = 'Test Email Subject',
	@profile_name = 'Test Mail Profile'

	--Send mail with attachment
	EXEC msdb.dbo.sp_send_dbmail
	@profile_name = 'Test Mail Profile'
	,@recipients = 'abc@xyz.gr'
	,@from_address = 'abc@xyz.com'
	,@execute_query_database = 'dbname'
	,@query = 'SELECT * FROM table'
	,@subject = 'Test Email Subject'
	,@body= 'Test Email Body'
	,@attach_query_result_as_file = 1
	,@query_attachment_filename = 'test.csv'
	,@query_result_separator = '	'
	,@query_result_no_padding= 1
	,@exclude_query_output =1
	,@append_query_error = 0
	,@query_result_header =1;
```
