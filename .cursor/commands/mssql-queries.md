# MS SQL Server Queries

## Description
Guide for writing efficient SQL queries in MS SQL Server including SELECT statements, JOINs, aggregations, CTEs, and window functions.

## Style Rules
- Use explicit JOIN syntax (not implicit comma joins)
- Alias tables with meaningful abbreviations
- Use column aliases for clarity
- Apply WHERE conditions before JOINs when possible
- Use EXISTS instead of IN for subqueries
- Add proper indexes for query optimization
- Use TOP or OFFSET-FETCH for pagination
- Format queries for readability

## Example: Basic SELECT

```sql
-- Select all columns
SELECT *
FROM [dbo].[User];

-- Select specific columns with aliases
SELECT 
    u.[Id] AS UserId,
    u.[Name] AS UserName,
    u.[Email],
    u.[CreatedAt]
FROM [dbo].[User] u
WHERE u.[IsActive] = 1
ORDER BY u.[CreatedAt] DESC;

-- Select with DISTINCT
SELECT DISTINCT u.[Email]
FROM [dbo].[User] u;

-- Select TOP N records
SELECT TOP 10
    u.[Id],
    u.[Name],
    u.[Email]
FROM [dbo].[User] u
ORDER BY u.[CreatedAt] DESC;
```

## Example: WHERE Clause

```sql
-- Simple conditions
SELECT *
FROM [dbo].[User] u
WHERE u.[IsActive] = 1
    AND u.[Email] LIKE '%@example.com'
    AND u.[CreatedAt] >= DATEADD(DAY, -30, GETUTCDATE());

-- IN clause
SELECT *
FROM [dbo].[User] u
WHERE u.[Id] IN (1, 2, 3, 4, 5);

-- BETWEEN clause
SELECT *
FROM [dbo].[Post] p
WHERE p.[CreatedAt] BETWEEN '2024-01-01' AND '2024-12-31';

-- NULL checks
SELECT *
FROM [dbo].[Post] p
WHERE p.[PublishedAt] IS NOT NULL;

-- Pattern matching
SELECT *
FROM [dbo].[User] u
WHERE u.[Name] LIKE 'John%'  -- Starts with John
    OR u.[Name] LIKE '%Doe'   -- Ends with Doe
    OR u.[Name] LIKE '%Smith%'; -- Contains Smith
```

## Example: JOINs

```sql
-- INNER JOIN (only matching records)
SELECT 
    u.[Name] AS AuthorName,
    p.[Title] AS PostTitle,
    p.[CreatedAt] AS PublishedDate
FROM [dbo].[User] u
INNER JOIN [dbo].[Post] p ON u.[Id] = p.[AuthorId]
WHERE p.[Status] = 'Published'
ORDER BY p.[CreatedAt] DESC;

-- LEFT JOIN (all records from left table)
SELECT 
    u.[Name],
    COUNT(p.[Id]) AS PostCount
FROM [dbo].[User] u
LEFT JOIN [dbo].[Post] p ON u.[Id] = p.[AuthorId]
GROUP BY u.[Id], u.[Name]
ORDER BY PostCount DESC;

-- Multiple JOINs
SELECT 
    u.[Name] AS AuthorName,
    p.[Title] AS PostTitle,
    c.[Content] AS CommentContent,
    cu.[Name] AS CommenterName
FROM [dbo].[Post] p
INNER JOIN [dbo].[User] u ON p.[AuthorId] = u.[Id]
INNER JOIN [dbo].[Comment] c ON p.[Id] = c.[PostId]
INNER JOIN [dbo].[User] cu ON c.[AuthorId] = cu.[Id]
WHERE p.[Status] = 'Published'
ORDER BY p.[CreatedAt] DESC, c.[CreatedAt] ASC;

-- Self JOIN
SELECT 
    e.[Name] AS EmployeeName,
    m.[Name] AS ManagerName
FROM [dbo].[Employee] e
LEFT JOIN [dbo].[Employee] m ON e.[ManagerId] = m.[Id];
```

## Example: Aggregations

```sql
-- Basic aggregations
SELECT 
    COUNT(*) AS TotalUsers,
    COUNT(DISTINCT [Email]) AS UniqueEmails,
    MAX([CreatedAt]) AS MostRecentUser,
    MIN([CreatedAt]) AS FirstUser
FROM [dbo].[User];

-- GROUP BY with HAVING
SELECT 
    u.[Name] AS Author,
    COUNT(p.[Id]) AS PostCount,
    MAX(p.[CreatedAt]) AS LastPostDate
FROM [dbo].[User] u
INNER JOIN [dbo].[Post] p ON u.[Id] = p.[AuthorId]
GROUP BY u.[Id], u.[Name]
HAVING COUNT(p.[Id]) > 5
ORDER BY PostCount DESC;

-- GROUP BY with multiple columns
SELECT 
    YEAR(p.[CreatedAt]) AS Year,
    MONTH(p.[CreatedAt]) AS Month,
    p.[Status],
    COUNT(*) AS PostCount
FROM [dbo].[Post] p
GROUP BY YEAR(p.[CreatedAt]), MONTH(p.[CreatedAt]), p.[Status]
ORDER BY Year DESC, Month DESC;
```

## Example: Subqueries

```sql
-- Subquery in WHERE
SELECT *
FROM [dbo].[User] u
WHERE u.[Id] IN (
    SELECT DISTINCT p.[AuthorId]
    FROM [dbo].[Post] p
    WHERE p.[Status] = 'Published'
);

-- Subquery with EXISTS (more efficient)
SELECT *
FROM [dbo].[User] u
WHERE EXISTS (
    SELECT 1
    FROM [dbo].[Post] p
    WHERE p.[AuthorId] = u.[Id]
        AND p.[Status] = 'Published'
);

-- Correlated subquery
SELECT 
    u.[Name],
    (SELECT COUNT(*)
     FROM [dbo].[Post] p
     WHERE p.[AuthorId] = u.[Id]) AS PostCount
FROM [dbo].[User] u;
```

## Example: Common Table Expressions (CTE)

```sql
-- Simple CTE
WITH ActiveUsers AS (
    SELECT 
        u.[Id],
        u.[Name],
        u.[Email]
    FROM [dbo].[User] u
    WHERE u.[IsActive] = 1
)
SELECT 
    au.[Name],
    COUNT(p.[Id]) AS PostCount
FROM ActiveUsers au
LEFT JOIN [dbo].[Post] p ON au.[Id] = p.[AuthorId]
GROUP BY au.[Id], au.[Name];

-- Multiple CTEs
WITH 
    PostStats AS (
        SELECT 
            p.[AuthorId],
            COUNT(*) AS PostCount,
            MAX(p.[CreatedAt]) AS LastPostDate
        FROM [dbo].[Post] p
        GROUP BY p.[AuthorId]
    ),
    CommentStats AS (
        SELECT 
            c.[AuthorId],
            COUNT(*) AS CommentCount
        FROM [dbo].[Comment] c
        GROUP BY c.[AuthorId]
    )
SELECT 
    u.[Name],
    ISNULL(ps.[PostCount], 0) AS Posts,
    ISNULL(cs.[CommentCount], 0) AS Comments
FROM [dbo].[User] u
LEFT JOIN PostStats ps ON u.[Id] = ps.[AuthorId]
LEFT JOIN CommentStats cs ON u.[Id] = cs.[AuthorId]
ORDER BY Posts DESC, Comments DESC;

-- Recursive CTE (for hierarchical data)
WITH EmployeeHierarchy AS (
    -- Anchor member
    SELECT 
        [Id],
        [Name],
        [ManagerId],
        0 AS Level
    FROM [dbo].[Employee]
    WHERE [ManagerId] IS NULL
    
    UNION ALL
    
    -- Recursive member
    SELECT 
        e.[Id],
        e.[Name],
        e.[ManagerId],
        eh.[Level] + 1
    FROM [dbo].[Employee] e
    INNER JOIN EmployeeHierarchy eh ON e.[ManagerId] = eh.[Id]
)
SELECT * FROM EmployeeHierarchy
ORDER BY [Level], [Name];
```

## Example: Window Functions

```sql
-- ROW_NUMBER for pagination
SELECT 
    ROW_NUMBER() OVER (ORDER BY p.[CreatedAt] DESC) AS RowNum,
    p.[Id],
    p.[Title],
    p.[CreatedAt]
FROM [dbo].[Post] p;

-- RANK and DENSE_RANK
SELECT 
    u.[Name],
    COUNT(p.[Id]) AS PostCount,
    RANK() OVER (ORDER BY COUNT(p.[Id]) DESC) AS Rank,
    DENSE_RANK() OVER (ORDER BY COUNT(p.[Id]) DESC) AS DenseRank
FROM [dbo].[User] u
LEFT JOIN [dbo].[Post] p ON u.[Id] = p.[AuthorId]
GROUP BY u.[Id], u.[Name];

-- PARTITION BY for grouped calculations
SELECT 
    p.[AuthorId],
    p.[Title],
    p.[CreatedAt],
    ROW_NUMBER() OVER (PARTITION BY p.[AuthorId] ORDER BY p.[CreatedAt] DESC) AS PostNumber,
    COUNT(*) OVER (PARTITION BY p.[AuthorId]) AS TotalPostsByAuthor
FROM [dbo].[Post] p;

-- LAG and LEAD for comparing rows
SELECT 
    p.[Title],
    p.[CreatedAt],
    LAG(p.[CreatedAt]) OVER (ORDER BY p.[CreatedAt]) AS PreviousPostDate,
    LEAD(p.[CreatedAt]) OVER (ORDER BY p.[CreatedAt]) AS NextPostDate,
    DATEDIFF(DAY, 
        LAG(p.[CreatedAt]) OVER (ORDER BY p.[CreatedAt]), 
        p.[CreatedAt]) AS DaysSincePrevious
FROM [dbo].[Post] p;
```

## Example: Pagination

```sql
-- Using OFFSET-FETCH (SQL Server 2012+)
DECLARE @PageNumber INT = 1;
DECLARE @PageSize INT = 10;

SELECT 
    u.[Id],
    u.[Name],
    u.[Email]
FROM [dbo].[User] u
ORDER BY u.[CreatedAt] DESC
OFFSET (@PageNumber - 1) * @PageSize ROWS
FETCH NEXT @PageSize ROWS ONLY;

-- Using ROW_NUMBER for older versions
WITH PagedResults AS (
    SELECT 
        ROW_NUMBER() OVER (ORDER BY u.[CreatedAt] DESC) AS RowNum,
        u.[Id],
        u.[Name],
        u.[Email]
    FROM [dbo].[User] u
)
SELECT *
FROM PagedResults
WHERE RowNum BETWEEN 11 AND 20; -- Page 2, 10 per page
```

## Example: UNION and UNION ALL

```sql
-- UNION (removes duplicates)
SELECT [Email] FROM [dbo].[User]
UNION
SELECT [Email] FROM [dbo].[ArchivedUser];

-- UNION ALL (keeps duplicates, faster)
SELECT [Email] FROM [dbo].[User]
UNION ALL
SELECT [Email] FROM [dbo].[ArchivedUser];

-- UNION with different sources
SELECT 
    'User' AS Source,
    COUNT(*) AS Count
FROM [dbo].[User]
UNION ALL
SELECT 
    'Post' AS Source,
    COUNT(*) AS Count
FROM [dbo].[Post]
UNION ALL
SELECT 
    'Comment' AS Source,
    COUNT(*) AS Count
FROM [dbo].[Comment];
```
