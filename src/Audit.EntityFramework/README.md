# Audit.EntityFramework

**EntityFramework (EF) Audit Extension for [Audit.NET library](https://github.com/thepirat000/Audit.NET).**

Automatically generates Audit Trails for EntityFramework's CRUD operations. **Supporting EF 6 and EF Core**.

Audit.EntityFramework provides the infrastructure to log interactions with the EF `DbContext`. It can record operations in the database with detailed information.

##Install

**[NuGet Package](https://www.nuget.org/packages/Audit.EntityFramework/)**
```
PM> Install-Package Audit.EntityFramework
```

##Usage
Change your EF Context class to inherits from `Audit.EntityFramework.AuditDbContext` instead of `DbContext`. For example if you have a context like this:

```c#
public class MyEntitites : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```

to enable the audit log, you should change it to this:
```c#
public class MyEntitites : Audit.EntityFramework.AuditDbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```

##Configuration
You can change the default behavior by decorating your DbContext with the `AuditDbContext` attribute, indicating the setting values:

- **Mode**: To indicate the audit operation mode
 - _Opt-Out_: All the entities are tracked by default, except those decorated with the `AuditIgnore` attribute. (Default)
 - _Opt-In_: No entity is tracked by default, except those decorated with the `AuditInclude` attribute.
- **IncludeEntityObjects**: To indicate if the output should contain the modified entities objects. (Default is false)
- **AuditEventType**: To indicate the event type to use on the audit event. (Default is the context name). Can contain the following placeholders: 
  - {context}: replaced with the Db Context type name.
  - {database}: replaced with the database name.

For example:
```c#
[AuditDbContext(Mode = AuditOptionMode.OptOut, IncludeEntityObjects = false, AuditEventType = "{database}_{context}" )]
public class MyEntitites : Audit.EntityFramework.AuditDbContext
{
...
```

You can also change the settings by accessing the properties with the same name as in the attribute. For example:
```c#
public class MyEntitites : Audit.EntityFramework.AuditDbContext
{
    public MyEntitites()
    {
        AuditEventType = "{database}_{context}";
        Mode = AuditOptionMode.OptOut;
        IncludeEntityObjects = false;
    }
}
```

##Output
Audit.EntityFramework output includes:
- Affected Database and Table names
- Affected column data including primary key, original and new values
- Model validation results
- Exception details
- Transaction information
- Entity object graphs (optional with `IncludeEntityObjects` configuration)
- Execution time and duration
- Enviroment information such as user, machine, domain, locale, etc.

With this information, you can not just know who did the operation, but also measure performance, observe exceptions thrown or get statistics about usage of your database.

##Output samples

This is an example of the output for a single Insert operation:
```javascript
{
	"EventType": "Blogs_MyContext",
	"Environment": {
		"UserName": "Federico",
		"MachineName": "HP",
		"DomainName": "HP",
		"CallingMethodName": "Audit.UnitTest.AuditTests.TestEF()",
		"Exception": null,
		"Culture": "en-GB"
	},
	"StartDate": "2016-09-06T21:11:57.7562152-05:00",
	"EndDate": "2016-09-06T21:11:58.1039904-05:00",
	"Duration": 348,
	"EntityFrameworkEvent": {
		"Database": "Blogs",
		"TransactionId": "620ad14e-ec3b-4eac-a8fe-cfe5e8f76e5d_1",
		"Entries": [{
			"EntityType": "Posts",
			"Action": "Add",
			"PrimaryKey": {
				"Id": 73
			},
			"Entity": {
				"Id": 73,
				"Title": "some title",
				"DateCreated": "2016-09-06T21:11:28.0257409-05:00",
				"Content": "some content",
				"BlogId": 1,
				"Blog": null
			},
			"Valid": true
		}],
		"Result": 1,
		"Success": true
	}
}
```
