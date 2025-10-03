# MS SQL Server Database Schema

## Description
Guide for creating database schemas in MS SQL Server with proper table design, constraints, indexes, and relationships.

## Style Rules
- Use singular table names with PascalCase (e.g., User, not Users)
- Define primary keys with IDENTITY for auto-increment
- Add foreign key constraints with proper naming
- Create indexes on frequently queried columns
- Use appropriate data types (NVARCHAR for Unicode text)
- Add DEFAULT constraints where appropriate
- Use CHECK constraints for data validation
- Add proper timestamps (CreatedAt, UpdatedAt)

## Example: Basic Table Creation

```sql
-- Create User table
CREATE TABLE [dbo].[User] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [Name] NVARCHAR(100) NOT NULL,
    [Email] NVARCHAR(255) NOT NULL,
    [PasswordHash] NVARCHAR(MAX) NOT NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [UpdatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CONSTRAINT [PK_User] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [UQ_User_Email] UNIQUE NONCLUSTERED ([Email] ASC),
    CONSTRAINT [CK_User_Name] CHECK (LEN([Name]) >= 2)
);
GO

-- Create index on Email for faster lookups
CREATE NONCLUSTERED INDEX [IX_User_Email]
    ON [dbo].[User] ([Email] ASC);
GO

-- Create index on IsActive for filtering
CREATE NONCLUSTERED INDEX [IX_User_IsActive]
    ON [dbo].[User] ([IsActive] ASC)
    WHERE [IsActive] = 1;
GO
```

## Example: Tables with Relationships

```sql
-- Create Post table
CREATE TABLE [dbo].[Post] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [Title] NVARCHAR(200) NOT NULL,
    [Content] NVARCHAR(MAX) NOT NULL,
    [AuthorId] INT NOT NULL,
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Draft',
    [PublishedAt] DATETIME2 NULL,
    [CreatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [UpdatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CONSTRAINT [PK_Post] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Post_User] FOREIGN KEY ([AuthorId])
        REFERENCES [dbo].[User] ([Id])
        ON DELETE CASCADE,
    CONSTRAINT [CK_Post_Status] CHECK ([Status] IN ('Draft', 'Published', 'Archived'))
);
GO

-- Create index on AuthorId for joins
CREATE NONCLUSTERED INDEX [IX_Post_AuthorId]
    ON [dbo].[Post] ([AuthorId] ASC);
GO

-- Create index on Status and PublishedAt for filtering
CREATE NONCLUSTERED INDEX [IX_Post_Status_PublishedAt]
    ON [dbo].[Post] ([Status] ASC, [PublishedAt] DESC);
GO

-- Create Comment table
CREATE TABLE [dbo].[Comment] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [Content] NVARCHAR(1000) NOT NULL,
    [PostId] INT NOT NULL,
    [AuthorId] INT NOT NULL,
    [CreatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CONSTRAINT [PK_Comment] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_Comment_Post] FOREIGN KEY ([PostId])
        REFERENCES [dbo].[Post] ([Id])
        ON DELETE CASCADE,
    CONSTRAINT [FK_Comment_User] FOREIGN KEY ([AuthorId])
        REFERENCES [dbo].[User] ([Id])
        ON DELETE NO ACTION
);
GO

-- Create indexes for foreign keys
CREATE NONCLUSTERED INDEX [IX_Comment_PostId]
    ON [dbo].[Comment] ([PostId] ASC);
GO

CREATE NONCLUSTERED INDEX [IX_Comment_AuthorId]
    ON [dbo].[Comment] ([AuthorId] ASC);
GO
```

## Example: Many-to-Many Relationship

```sql
-- Create Tag table
CREATE TABLE [dbo].[Tag] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [Name] NVARCHAR(50) NOT NULL,
    [CreatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CONSTRAINT [PK_Tag] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [UQ_Tag_Name] UNIQUE NONCLUSTERED ([Name] ASC)
);
GO

-- Create junction table for Post-Tag relationship
CREATE TABLE [dbo].[PostTag] (
    [PostId] INT NOT NULL,
    [TagId] INT NOT NULL,
    [CreatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CONSTRAINT [PK_PostTag] PRIMARY KEY CLUSTERED ([PostId] ASC, [TagId] ASC),
    CONSTRAINT [FK_PostTag_Post] FOREIGN KEY ([PostId])
        REFERENCES [dbo].[Post] ([Id])
        ON DELETE CASCADE,
    CONSTRAINT [FK_PostTag_Tag] FOREIGN KEY ([TagId])
        REFERENCES [dbo].[Tag] ([Id])
        ON DELETE CASCADE
);
GO

-- Create indexes for foreign keys
CREATE NONCLUSTERED INDEX [IX_PostTag_PostId]
    ON [dbo].[PostTag] ([PostId] ASC);
GO

CREATE NONCLUSTERED INDEX [IX_PostTag_TagId]
    ON [dbo].[PostTag] ([TagId] ASC);
GO
```

## Example: Audit Table

```sql
-- Create AuditLog table for tracking changes
CREATE TABLE [dbo].[AuditLog] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [TableName] NVARCHAR(50) NOT NULL,
    [RecordId] INT NOT NULL,
    [Action] NVARCHAR(10) NOT NULL,
    [OldValues] NVARCHAR(MAX) NULL,
    [NewValues] NVARCHAR(MAX) NULL,
    [ChangedBy] INT NOT NULL,
    [ChangedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CONSTRAINT [PK_AuditLog] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_AuditLog_User] FOREIGN KEY ([ChangedBy])
        REFERENCES [dbo].[User] ([Id]),
    CONSTRAINT [CK_AuditLog_Action] CHECK ([Action] IN ('INSERT', 'UPDATE', 'DELETE'))
);
GO

-- Create index for querying audit logs
CREATE NONCLUSTERED INDEX [IX_AuditLog_TableName_RecordId]
    ON [dbo].[AuditLog] ([TableName] ASC, [RecordId] ASC);
GO

CREATE NONCLUSTERED INDEX [IX_AuditLog_ChangedAt]
    ON [dbo].[AuditLog] ([ChangedAt] DESC);
GO
```

## Example: Computed Columns

```sql
-- Create Product table with computed column
CREATE TABLE [dbo].[Product] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [Name] NVARCHAR(100) NOT NULL,
    [Price] DECIMAL(10, 2) NOT NULL,
    [DiscountPercent] DECIMAL(5, 2) NOT NULL DEFAULT 0,
    [FinalPrice] AS ([Price] * (1 - [DiscountPercent] / 100)) PERSISTED,
    [CreatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    CONSTRAINT [PK_Product] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [CK_Product_Price] CHECK ([Price] > 0),
    CONSTRAINT [CK_Product_Discount] CHECK ([DiscountPercent] >= 0 AND [DiscountPercent] <= 100)
);
GO
```

## Example: Full-Text Search

```sql
-- Enable full-text search on Post table
CREATE FULLTEXT CATALOG [ftCatalog] AS DEFAULT;
GO

CREATE FULLTEXT INDEX ON [dbo].[Post] (
    [Title],
    [Content]
)
KEY INDEX [PK_Post]
WITH STOPLIST = SYSTEM;
GO
```

## Example: Schema Creation

```sql
-- Create a schema for organizing related tables
CREATE SCHEMA [Blog] AUTHORIZATION [dbo];
GO

-- Move tables to schema
ALTER SCHEMA [Blog] TRANSFER [dbo].[Post];
ALTER SCHEMA [Blog] TRANSFER [dbo].[Comment];
ALTER SCHEMA [Blog] TRANSFER [dbo].[Tag];
ALTER SCHEMA [Blog] TRANSFER [dbo].[PostTag];
GO

-- Now tables can be accessed as [Blog].[Post], etc.
```

## Example: Temporal Table (System-Versioned)

```sql
-- Create temporal table for tracking history
CREATE TABLE [dbo].[UserProfile] (
    [Id] INT IDENTITY(1,1) NOT NULL,
    [UserId] INT NOT NULL,
    [Bio] NVARCHAR(500) NULL,
    [Website] NVARCHAR(200) NULL,
    [ValidFrom] DATETIME2 GENERATED ALWAYS AS ROW START NOT NULL,
    [ValidTo] DATETIME2 GENERATED ALWAYS AS ROW END NOT NULL,
    PERIOD FOR SYSTEM_TIME ([ValidFrom], [ValidTo]),
    CONSTRAINT [PK_UserProfile] PRIMARY KEY CLUSTERED ([Id] ASC),
    CONSTRAINT [FK_UserProfile_User] FOREIGN KEY ([UserId])
        REFERENCES [dbo].[User] ([Id])
        ON DELETE CASCADE
)
WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = [dbo].[UserProfileHistory]));
GO
```

## Drop Tables (in correct order)

```sql
-- Drop tables in reverse order of dependencies
DROP TABLE IF EXISTS [dbo].[Comment];
DROP TABLE IF EXISTS [dbo].[PostTag];
DROP TABLE IF EXISTS [dbo].[Tag];
DROP TABLE IF EXISTS [dbo].[Post];
DROP TABLE IF EXISTS [dbo].[AuditLog];
DROP TABLE IF EXISTS [dbo].[User];
DROP TABLE IF EXISTS [dbo].[Product];
GO
```
