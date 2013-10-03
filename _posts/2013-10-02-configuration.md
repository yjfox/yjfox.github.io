---
layout: default
title: Configuration of PHP+Apache+MySQL+MSSQL+Neo4j
label: Java, objective
time: 2013-10-02
brief: Instruction of installation and configuration of a complicated server which support PHP+Apache+MySQL+MSSQL+Neo4j and also JAVA.
---
#{{page.title}}
> Instruction of installation and configuration of a complicated server which support PHP+Apache+MySQL+MSSQL+Neo4j and also JAVA.
**************

##Development Environment Setup

- phpAdmin:

1.1 Install either WAMP (for windows) or MAMP (for Mac) or LAMP (for Linux) depending on your OS if you don't have Apache, MySQL and PHP on your machine.  
P.S:If you use WAMP. Please install 32 bit version even your system is 64 bit, because the driver of sqlsrv only provides 32 bit version.

1.2 Check:  
After click "Start All Services" and the icon turns green, then everything is ok. If the icon turns orange, then means some other process is using the portal. Solution: Test your port 80 from wamp->Apache->service, if your default port:80 is used by Microsoft. Please go to control panel->Administrative Tools->Services. Stop your SQL Server Reporting Services and set the start type to manual.

- Create a Colfusion folder (it might be case sensitive, so please type capital C) in the web directory. Folder:

LAMP: \var\www  
WAMP: \wamp\www  
MAMP: \Applications\MAMP\htdocs

- Check out (using SVN) the source code into the Colfusion folder.

3.1 Download SVN. Recommend download link  
3.2 Go to Colfusion folder, right click on the folder and select "SVN check out".  
3.3 Use the URL (https://tycho.exp.sis.pitt.edu/svn/Colfusion/trunk) and login and password which you receive in the email from Evgeny.  
3.4 Check:  
Make sure that after check out you don't have another Colfusion folder inside of the created Colfusion folder. At the end you should have one Colfusion folder which contains source code (many folders and php files).  

- Import the database in your local mysql server.

4.1 Go to database_scripts folder under Colfusion folder and find colfusion_schema_and_config_data.sql file, then open this file as a txt file and copy everything of it.  
4.2 Click WAMPSERVER icon and select "phpMyAdmin".  
4.3 Login with username (root) and without password.  
4.4 Create Colfusion database in your local mysql server.  
4.5 Open SQL tab and paste content into the textbox.  
4.6 Click "GO".  
4.7 Import all other database script found in the database_scripts folder except for geonames and clean_data_source.sql.  
4.8 In User tab, add "dataverse" mysql user with "dataverse" password. Make user have all privileges. Host choose "Local".  
4.9 In User tab, add "ImportTester" mysql user with "importtester" password. Make user have all privileges. Host choose "Local".  

- Configure dbconnect to connect to your local mysql. 

5.1 Most probably you don't need to change anything.  
5.2 Go to Colfusion/libs/dbconnect.php file and provide information to connect to your local mysql. Most probably you will only need to modify EZSQL_DB_USER and EZSQL_DB_PASSWORD.    

**The first part is done***********************************************  

Check: you should be able to go to your favorite web browser and type localhost/Colfusion and the web site.  

- Install MS SQL server 2008 R2 Developer. If you have linux on mac you would need to set up virtual machine with windows on it.

7.1 Download Link (sorry, I cannot provide)  
7.2 At first you need to register on that web site and get verified that you are a student at Pitt. Then you will be able to download it.  
7.3 Install MS SQL server 2008 R2 Developer.  
7.4 Open MS SQL server Management Studio, enter "(local)" in Server name, select "Windows Authentication" in Authentication, then click "Connect".

- Add a new user (remoteUserTest) which will have remote access to the server.

8.1 Select Security and right click "Logins" on the left side and select "New Login...".  
8.2 Enter Login name, select "SQL Server authentication", enter password.  
8.3 Select "Server Roles" tab, click "sysadmin" privileges, then click OK.  
8.4 Check:  
The new user can be connected correctly. First, click "disconnect" icon on the top of left side, then click "connect Object Explorer" icon beside "disconnect". Use "SQL Server Authentication" as Authentication and enter remoteUserTest and password. If it failed, then modify something in SQL server configuration management. (I cannot remember clearly here.)

- Create databases in MS SQL.

9.1 Create a database for cached queries. I suggest you named the database "colfusion_cached_queries".
9.2 Create a database for cached distinct values for data matching. I suggest you name it "colfusion_relationships_columns".
9.3 Select just created database (colfusion_relationships_columns) and run following query:

{% highlight sql %}
    USE [colfusion_relationships_columns] 
    GO

    SET ANSI_NULLS ON
    GO

    SET QUOTED_IDENTIFIER ON
    GO

    CREATE TABLE [dbo].[transformationDistinctValue](
    [transformation] [nvarchar](400) NOT NULL,
    [value] [nvarchar](400) NOT NULL
    ) ON [PRIMARY]

    GO

    CREATE NONCLUSTERED INDEX [IX_transformationDistinctValue] ON [dbo].[transformationDistinctValue] 
    (
    [transformation] ASC
    )WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, IGNORE_DUP_KEY = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
    GO
{% endhighlight %}

9.4 Install MySQL odbc connector. Download Link (cannot provide)

- Add linked server for colfusion.

10.1 Create new query and execute it:

{% highlight sql %}
    exec sp_addlinkedserver
    @server = N'colfusion',
    @srvproduct = N'MySQL',
    @provider = N'MSDASQL',
	@provstr =N'DRIVER={MySQL ODBC 5.2 Unicode Driver}; SERVER=localhost;PORT=3306;DATABASE=colfusion;USER=dataverse;PASSWORD=dataverse;OPTION=3;';

	exec sp_addlinkedsrvlogin
	@rmtsrvname = N'colfusion',
	@locallogin = N'remoteUserTest',
	@rmtuser = N'dataverse',
	@rmtpassword = N'dataverse',
	@useself=N'False';
{% endhighlight %}

10.2 Select "Server Objects - Linked Servers - Providers" on the left side, then right click "MSDASQL", select "Properties".   
10.3 Set "Nested queries", "Level zero only" and "Supports 'Like' operator" for it.  
10.4 Check:   
Select "Server Objects - Linked Servers - Providers" on the left side, then right click "colfusion".  
10.5 Select "Test Connection", then a dialog "The test connection to the linked server succeeded" will appear.  
10.5_sub: Once you got an error message like, "Cannot create an instance of OLE DB provider "provider name" for linked server "linked server name". (Microsoft SQL Server, Error: 7302)". Try these things. 1, Check whether you can log in MySql with "user: dataverse; password: dataverse". If you can't, add several "dataverse" with different "Host", such as "127.0.0.1", "localhost", "%". Just make sure that you can login with "dataverse".  
10.6 Download attached php files and put them into folder "Colfusion".  
10.7 Go to Colfusion/DAL/CachedServersCred.php file and provide information of the remote user for the MS SQL server, the server name (localhost) and the database which you have created. Most probably you will not to change anything, except that you need to provide the password which you set up for the user.  
10.8 Go to Colfusion/DAL/LinkedServerCred.php and provide the same information. Most probably you will not to change anything, except that you need to provide the password which you set up for the user.  
10.9 Check:  
Test your port 80 from wamp->Apache->service, if your default port:80 is used by Microsoft. Please go to control panel->Administrative Tools->Services. Stop your SQL Server Reporting Services and set the start type to manual  
10.10 Check:  
To make submission of dataset as database dump file work, you will need to edit Colfusion/DAL/DBImporters/DatabaseImporterFactory.php file and provide information about the connection information for all databases listed there. Most probably you don't need to make any changes.  
10.11 Check:  
Make sure you never commit the files mentioned above: dbconnect.php, CachedServersCred.php, LinkedServerCred.php, DatabaseImporterFactory.php

**Second Part done***********************************************  

- Installation of Neo4j

11.1 search for neo4j for windows, download the official version (either community or advanced is OK)  
11.2 make sure you have installed the JDK (download one if you dont have), then check the user's path: right click "computer", there is a user's path configuration  
11.3 open neo4j folder and get into bin directory, run the document neo4j.bat  
11.4 type "localhost:[port]" in your favorite browser. If everything is good, neo4J would appear.

**The most important part:***********************************************  

12.1 As the neo4j requires "curl" command, which only available in linux/Unix, so we need modify the php.ini to activate the "curl"  
12.2 make sure your phpAdmin is running, type "localhost/?phpini" in browser, search for key word of "curl", find out where it is.  
12.3 according to what you found in previous step, go find the php.ini document and search for key word "curl" again.  
12.4 if you still catch me, there should be a sentence containing something like "php.curl", and I guess it has been commented.  
12.5 get off the comment syntax ";", activate this sentence

**All steps done here, enjoy your development environment with everything you need**
