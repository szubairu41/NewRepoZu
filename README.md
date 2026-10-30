USE ZEN_ZENITH_PRD;
GO
SET NOCOUNT ON;

SELECT s.session_id,
       s.login_name,
       s.host_name,
       s.program_name,
       s.status,                               -- 'sleeping' = connected but idle
       s.login_time,
       s.last_request_end_time,
       DATEDIFF(MINUTE, s.last_request_end_time, GETDATE()) AS IdleMinutes,
       s.open_transaction_count
FROM sys.dm_exec_sessions s
WHERE s.is_user_process = 1
  AND s.database_id = DB_ID('ZEN_ZENITH_PRD')
  AND s.session_id <> @@SPID
ORDER BY s.last_request_end_time DESC;
