# SAMPLE_Changed-entities-form
Setup package to track entity changes in sysHUB and display them in a custom form of the webclient

## Setup

### Database changes

#### 1. Create tracking table

```sql
DROP TABLE IF EXISTS [dbo].[_changes]
GO

CREATE TABLE [dbo].[_changes](
	[id] [bigint] IDENTITY(1,1) NOT NULL,
	[modifiedby] [nvarchar](32) NULL,
	[uuid] [nvarchar](32) NOT NULL,
	[modifiedtime] [datetime] NOT NULL,
	[type] [nvarchar](16) NOT NULL,
	[name_before] [nvarchar](64) NOT NULL,
	[name_after] [nvarchar](64) NULL,
	[value_before] [nvarchar](2048) NULL,
	[value_after] [nvarchar](2048) NULL,
	[type_before] [int] NULL,
	[type_after] [int] NULL,
	[description_before] [nvarchar](256) NULL,
	[description_after] [nvarchar](256) NULL,
	[parentuuid_before] [nvarchar](32) NULL,
	[parentuuid_after] [nvarchar](50) NULL,
	[bindata] [varbinary](max) NULL,
	[strdata] [varchar](max) NULL,
 CONSTRAINT [PK__changes] PRIMARY KEY CLUSTERED 
(
	[id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
```

#### 2. Create trigger for entities: table `category`

```sql
DROP TRIGGER IF EXISTS [dbo].[category_AfterUpdate]
GO

CREATE TRIGGER [dbo].[category_AfterUpdate]
   ON [dbo].[category]
   AFTER UPDATE
AS 
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;
	INSERT INTO dbo._changes(modifiedby, modifiedtime, type, uuid, name_before, name_after, description_before, description_after)
		SELECT i.modifiedby, GETDATE(), 'category', i.uuid, d.name, i.name, d.description, i.description
		FROM Inserted i
		INNER JOIN Deleted d on i.uuid = d.uuid;
END
GO

ALTER TABLE [dbo].[category] ENABLE TRIGGER [category_AfterUpdate]
GO
```

#### 2. Create trigger for entities: table `config`

```sql
DROP TRIGGER IF EXISTS [dbo].[config_AfterUpdate]
GO

CREATE TRIGGER [dbo].[config_AfterUpdate]
   ON [dbo].[config]
   AFTER UPDATE
AS 
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;
	INSERT INTO dbo._changes(modifiedby, modifiedtime, type, uuid, name_before, name_after, value_before, value_after, type_before, type_after, description_before, description_after, parentuuid_before, parentuuid_after)
		SELECT i.modifiedby, GETDATE(), 'config', i.uuid, d.config_name, i.config_name, d.config_value, i.config_value, d.config_type, i.config_type, d.description, i.description, d.parentuuid, i.parentuuid
		FROM Inserted i
		INNER JOIN Deleted d on i.uuid = d.uuid;
END
GO

ALTER TABLE [dbo].[config] ENABLE TRIGGER [config_AfterUpdate]
GO
```



