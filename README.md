USE ZEN_ZENITH;
GO
SET NOCOUNT ON;

-- 1. Who is connected to ZEN_ZENITH right now
--    (without VIEW SERVER STATE you'll only see your own session)
SELECT s.login_name,
       s.host_name,
       s.program_name,
       COUNT(*)                AS SessionCount,
       MAX(s.last_request_end_time) AS LastActivity,
       MAX(s.login_time)       AS MostRecentLogin
FROM sys.dm_exec_sessions s
WHERE s.is_user_process = 1
  AND s.database_id = DB_ID('ZEN_ZENITH')
GROUP BY s.login_name, s.host_name, s.program_name
ORDER BY LastActivity DESC;

-- 2. Who has an actual request running this second
SELECT s.login_name, s.host_name, r.status, r.command,
       r.start_time, DB_NAME(r.database_id) AS DatabaseName
FROM sys.dm_exec_requests r
JOIN sys.dm_exec_sessions s ON s.session_id = r.session_id
WHERE s.is_user_process = 1
  AND r.database_id = DB_ID('ZEN_ZENITH')
  AND r.session_id <> @@SPID;

-- 3. Who has access to the database (users, not activity)
SELECT dp.name AS DatabaseUser,
       dp.type_desc,
       dp.create_date,
       dp.default_schema_name,
       STRING_AGG(r.name, ', ') AS DatabaseRoles
FROM sys.database_principals dp
LEFT JOIN sys.database_role_members drm ON drm.member_principal_id = dp.principal_id
LEFT JOIN sys.database_principals r     ON r.principal_id = drm.role_principal_id
WHERE dp.type IN ('S', 'U', 'G')
  AND dp.name NOT IN ('dbo', 'guest', 'INFORMATION_SCHEMA', 'sys')
GROUP BY dp.name, dp.type_desc, dp.create_date, dp.default_schema_name
ORDER BY dp.name;
