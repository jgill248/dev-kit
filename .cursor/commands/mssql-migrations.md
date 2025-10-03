# MS SQL Server Migrations

## Description
Guide for managing database schema changes in MS SQL Server through migrations, including adding/modifying tables, columns, indexes, and data transformations.

## Style Rules
- Always backup database before migrations
- Use transactions for rollback capability
- Test migrations on development environment first
- Include both UP and DOWN migration scripts
- Document breaking changes clearly
- Handle existing data appropriately
- Use IF EXISTS checks to make scripts idempotent
- Version migration scripts chronologically

## Example: Add New Column

```sql
-- Migration: Add PhoneNumber column to User table
-- Version: 2024-01-15-001
-- Description: Add optional phone number field

BEGIN TRANSACTION;
GO

-- Check if column already exists
IF NOT EXISTS (
    SELECT 1 
    FROM sys.columns 
    WHERE object_id = OBJECT_ID('[dbo].[User]') 
        AND name = 'PhoneNumber'
)
BEGIN
    ALTER TABLE [dbo].[User]
    ADD [PhoneNumber] NVARCHAR(20) NULL;
    
    PRINT 'Added PhoneNumber column to User table';
END
ELSE
BEGIN
    PRINT 'PhoneNumber column already exists';
END
GO

COMMIT TRANSACTION;
GO

-- Rollback script
/*
BEGIN TRANSACTION;
GO

IF EXISTS (
    SELECT 1 
    FROM sys.columns 
    WHERE object_id = OBJECT_ID('[dbo].[User]') 
        AND name = 'PhoneNumber'
)
BEGIN
    ALTER TABLE [dbo].[User]
    DROP COLUMN [PhoneNumber];
    
    PRINT 'Removed PhoneNumber column from User table';
END
GO

COMMIT TRANSACTION;
GO
*/
```

## Example: Modify Column

```sql
-- Migration: Change Email column to increase size
-- Version: 2024-01-16-001
-- Description: Increase Email field size from 255 to 320

BEGIN TRANSACTION;
GO

-- Alter column type
ALTER TABLE [dbo].[User]
ALTER COLUMN [Email] NVARCHAR(320) NOT NULL;

PRINT 'Modified Email column size to 320';
GO

COMMIT TRANSACTION;
GO

-- Rollback script
/*
BEGIN TRANSACTION;
GO

-- Verify no emails exceed 255 characters
IF NOT EXISTS (
    SELECT 1 FROM [dbo].[User] WHERE LEN([Email]) > 255
)
BEGIN
    ALTER TABLE [dbo].[User]
    ALTER COLUMN [Email] NVARCHAR(255) NOT NULL;
    
    PRINT 'Reverted Email column size to 255';
END
ELSE
BEGIN
    PRINT 'ERROR: Cannot revert - emails exist longer than 255 characters';
    ROLLBACK TRANSACTION;
END
GO

COMMIT TRANSACTION;
GO
*/
```

## Example: Add New Table

```sql
-- Migration: Create UserProfile table
-- Version: 2024-01-17-001
-- Description: Add user profile information table

BEGIN TRANSACTION;
GO

IF NOT EXISTS (
    SELECT 1 
    FROM sys.tables 
    WHERE name = 'UserProfile' AND schema_id = SCHEMA_ID('dbo')
)
BEGIN
    CREATE TABLE [dbo].[UserProfile] (
        [Id] INT IDENTITY(1,1) NOT NULL,
        [UserId] INT NOT NULL,
        [Bio] NVARCHAR(500) NULL,
        [Website] NVARCHAR(200) NULL,
        [Location] NVARCHAR(100) NULL,
        [CreatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
        [UpdatedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
        CONSTRAINT [PK_UserProfile] PRIMARY KEY CLUSTERED ([Id] ASC),
        CONSTRAINT [FK_UserProfile_User] FOREIGN KEY ([UserId])
            REFERENCES [dbo].[User] ([Id])
            ON DELETE CASCADE,
        CONSTRAINT [UQ_UserProfile_UserId] UNIQUE ([UserId])
    );
    
    CREATE NONCLUSTERED INDEX [IX_UserProfile_UserId]
        ON [dbo].[UserProfile] ([UserId] ASC);
    
    PRINT 'Created UserProfile table';
END
ELSE
BEGIN
    PRINT 'UserProfile table already exists';
END
GO

COMMIT TRANSACTION;
GO

-- Rollback script
/*
BEGIN TRANSACTION;
GO

IF EXISTS (
    SELECT 1 
    FROM sys.tables 
    WHERE name = 'UserProfile' AND schema_id = SCHEMA_ID('dbo')
)
BEGIN
    DROP TABLE [dbo].[UserProfile];
    PRINT 'Dropped UserProfile table';
END
GO

COMMIT TRANSACTION;
GO
*/
```

## Example: Add Index

```sql
-- Migration: Add index to Post table for better query performance
-- Version: 2024-01-18-001
-- Description: Add composite index on Status and CreatedAt

BEGIN TRANSACTION;
GO

IF NOT EXISTS (
    SELECT 1 
    FROM sys.indexes 
    WHERE name = 'IX_Post_Status_CreatedAt' 
        AND object_id = OBJECT_ID('[dbo].[Post]')
)
BEGIN
    CREATE NONCLUSTERED INDEX [IX_Post_Status_CreatedAt]
        ON [dbo].[Post] ([Status] ASC, [CreatedAt] DESC)
        INCLUDE ([Title], [AuthorId]);
    
    PRINT 'Created index IX_Post_Status_CreatedAt';
END
ELSE
BEGIN
    PRINT 'Index IX_Post_Status_CreatedAt already exists';
END
GO

COMMIT TRANSACTION;
GO

-- Rollback script
/*
BEGIN TRANSACTION;
GO

IF EXISTS (
    SELECT 1 
    FROM sys.indexes 
    WHERE name = 'IX_Post_Status_CreatedAt' 
        AND object_id = OBJECT_ID('[dbo].[Post]')
)
BEGIN
    DROP INDEX [IX_Post_Status_CreatedAt] ON [dbo].[Post];
    PRINT 'Dropped index IX_Post_Status_CreatedAt';
END
GO

COMMIT TRANSACTION;
GO
*/
```

## Example: Data Migration

```sql
-- Migration: Migrate user roles from text to foreign key
-- Version: 2024-01-19-001
-- Description: Create Role table and migrate existing role data

BEGIN TRANSACTION;
GO

-- Step 1: Create Role table
IF NOT EXISTS (
    SELECT 1 FROM sys.tables 
    WHERE name = 'Role' AND schema_id = SCHEMA_ID('dbo')
)
BEGIN
    CREATE TABLE [dbo].[Role] (
        [Id] INT IDENTITY(1,1) NOT NULL,
        [Name] NVARCHAR(50) NOT NULL,
        CONSTRAINT [PK_Role] PRIMARY KEY CLUSTERED ([Id] ASC),
        CONSTRAINT [UQ_Role_Name] UNIQUE ([Name])
    );
    
    -- Insert standard roles
    INSERT INTO [dbo].[Role] ([Name])
    VALUES ('Admin'), ('User'), ('Guest');
    
    PRINT 'Created Role table and inserted standard roles';
END
GO

-- Step 2: Add RoleId column to User table
IF NOT EXISTS (
    SELECT 1 FROM sys.columns 
    WHERE object_id = OBJECT_ID('[dbo].[User]') AND name = 'RoleId'
)
BEGIN
    ALTER TABLE [dbo].[User]
    ADD [RoleId] INT NULL;
    
    PRINT 'Added RoleId column to User table';
END
GO

-- Step 3: Migrate data from Role string to RoleId
IF EXISTS (
    SELECT 1 FROM sys.columns 
    WHERE object_id = OBJECT_ID('[dbo].[User]') AND name = 'Role'
)
BEGIN
    UPDATE u
    SET u.[RoleId] = r.[Id]
    FROM [dbo].[User] u
    INNER JOIN [dbo].[Role] r ON u.[Role] = r.[Name];
    
    -- Set default role for users without a valid role
    UPDATE [dbo].[User]
    SET [RoleId] = (SELECT [Id] FROM [dbo].[Role] WHERE [Name] = 'User')
    WHERE [RoleId] IS NULL;
    
    PRINT 'Migrated role data';
END
GO

-- Step 4: Make RoleId NOT NULL and add foreign key
ALTER TABLE [dbo].[User]
ALTER COLUMN [RoleId] INT NOT NULL;
GO

IF NOT EXISTS (
    SELECT 1 FROM sys.foreign_keys 
    WHERE name = 'FK_User_Role'
)
BEGIN
    ALTER TABLE [dbo].[User]
    ADD CONSTRAINT [FK_User_Role] FOREIGN KEY ([RoleId])
        REFERENCES [dbo].[Role] ([Id]);
    
    PRINT 'Added foreign key constraint';
END
GO

-- Step 5: Drop old Role column
IF EXISTS (
    SELECT 1 FROM sys.columns 
    WHERE object_id = OBJECT_ID('[dbo].[User]') AND name = 'Role'
)
BEGIN
    ALTER TABLE [dbo].[User]
    DROP COLUMN [Role];
    
    PRINT 'Dropped old Role column';
END
GO

COMMIT TRANSACTION;
GO
```

## Example: Rename Table/Column

```sql
-- Migration: Rename Comment table to PostComment
-- Version: 2024-01-20-001
-- Description: Rename Comment table for clarity

BEGIN TRANSACTION;
GO

IF EXISTS (
    SELECT 1 FROM sys.tables 
    WHERE name = 'Comment' AND schema_id = SCHEMA_ID('dbo')
)
BEGIN
    EXEC sp_rename '[dbo].[Comment]', 'PostComment';
    PRINT 'Renamed Comment table to PostComment';
END
GO

-- Rename column
IF EXISTS (
    SELECT 1 FROM sys.columns 
    WHERE object_id = OBJECT_ID('[dbo].[User]') AND name = 'Name'
)
BEGIN
    EXEC sp_rename '[dbo].[User].[Name]', 'FullName', 'COLUMN';
    PRINT 'Renamed Name column to FullName';
END
GO

COMMIT TRANSACTION;
GO

-- Rollback script
/*
BEGIN TRANSACTION;
GO

IF EXISTS (
    SELECT 1 FROM sys.tables 
    WHERE name = 'PostComment' AND schema_id = SCHEMA_ID('dbo')
)
BEGIN
    EXEC sp_rename '[dbo].[PostComment]', 'Comment';
    PRINT 'Reverted table name to Comment';
END
GO

IF EXISTS (
    SELECT 1 FROM sys.columns 
    WHERE object_id = OBJECT_ID('[dbo].[User]') AND name = 'FullName'
)
BEGIN
    EXEC sp_rename '[dbo].[User].[FullName]', 'Name', 'COLUMN';
    PRINT 'Reverted column name to Name';
END
GO

COMMIT TRANSACTION;
GO
*/
```

## Example: Add Constraint

```sql
-- Migration: Add check constraint to User table
-- Version: 2024-01-21-001
-- Description: Ensure email ends with valid domain

BEGIN TRANSACTION;
GO

IF NOT EXISTS (
    SELECT 1 FROM sys.check_constraints 
    WHERE name = 'CK_User_Email_Domain'
)
BEGIN
    ALTER TABLE [dbo].[User]
    ADD CONSTRAINT [CK_User_Email_Domain]
    CHECK ([Email] LIKE '%@%.%');
    
    PRINT 'Added email domain check constraint';
END
ELSE
BEGIN
    PRINT 'Constraint CK_User_Email_Domain already exists';
END
GO

COMMIT TRANSACTION;
GO

-- Rollback script
/*
BEGIN TRANSACTION;
GO

IF EXISTS (
    SELECT 1 FROM sys.check_constraints 
    WHERE name = 'CK_User_Email_Domain'
)
BEGIN
    ALTER TABLE [dbo].[User]
    DROP CONSTRAINT [CK_User_Email_Domain];
    
    PRINT 'Removed email domain check constraint';
END
GO

COMMIT TRANSACTION;
GO
*/
```

## Example: Migration Version Tracking

```sql
-- Create migration tracking table
IF NOT EXISTS (
    SELECT 1 FROM sys.tables 
    WHERE name = 'MigrationHistory' AND schema_id = SCHEMA_ID('dbo')
)
BEGIN
    CREATE TABLE [dbo].[MigrationHistory] (
        [Id] INT IDENTITY(1,1) NOT NULL,
        [Version] NVARCHAR(50) NOT NULL,
        [Description] NVARCHAR(500) NOT NULL,
        [AppliedAt] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
        CONSTRAINT [PK_MigrationHistory] PRIMARY KEY CLUSTERED ([Id] ASC),
        CONSTRAINT [UQ_MigrationHistory_Version] UNIQUE ([Version])
    );
    
    PRINT 'Created MigrationHistory table';
END
GO

-- Record migration
INSERT INTO [dbo].[MigrationHistory] ([Version], [Description])
VALUES ('2024-01-21-001', 'Add email domain check constraint');
GO
```
