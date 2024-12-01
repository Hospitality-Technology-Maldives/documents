![banner](/img/Oracle-VS-Code-SQL-Developer-Social-Media.png)

---
# VS CODE Oracle Database Connection

### Installation

1. Open Visual Studio Code.
2. Go to the Extensions view and install the following extension:

   - **Name:** [Oracle Developer Tools for VS Code (SQL and PLSQL)](https://marketplace.visualstudio.com/items?itemName=Oracle.oracledevtools)
   - **ID:** Oracle.oracledevtools
   - **Description:** Develop SQL and PL/SQL with Oracle Database and Oracle Autonomous Database
   - **Version:** 23.4.0
   - **Publisher:** Oracle Corporation

For a comprehensive guide on getting started, visit the [Oracle Developer Tools for VS Code Documentation](https://docs.oracle.com/en/database/oracle/developer-tools-for-vscode/getting-started/index.html).

### Connecting to the Database

Once the extension is installed and Visual Studio Code is restarted, a new `Oracle Explorer` will be available. To connect to the database:

1. In the Oracle Explorer, click the plus (+) sign next to the database.

#### Connecting Using Host Name/IP Address and Service Name

- **Connection Type:** Basic (Host, Port, Service Name)
- **Database Host Name:** Opera DB server IP or Hostname
- **Port Number:** Typically `1521`
- **Service Name:** The database service name, usually `OPERA`
- **Role:** Default (for ordinary use) or SYSDBA (for DBA tasks like creating a user)
- **Username:** Enter the username and password for the Opera schema.
- **Save Password:** Check this box if you do not wish to enter the password each time.
- **Connection Name:** Provide a connection name for reference in Database Explorer and elsewhere.

Click the "Create Connection" button.

> [!NOTE]
> Connection information and settings can be stored at the User, Workspace, or Folder level. For more details, see the documentation on [Organizing Connections](https://docs.oracle.com/en/database/oracle/developer-tools-for-vscode/getting-started/organizing-connections.html).\_

#### Connecting Using TNSNAMES.ORA File

If you have an Oracle client installed, you can use a `tnsnames.ora` file:

1. Copy the `tnsnames.ora` file into the directory set as the Config Files Folder in the Oracle Developer Tools for VS Code Extension Settings. By default, this location is `~/Oracle/network/admin` on Linux and Mac, and `%USERPROFILE%\Oracle\network\admin` on Windows.
2. If you need to create a `tnsnames.ora` file, refer to the sample located at `%USERPROFILE%\.vscode\extensions\oracle.oracledevtools-<version>\sample\network` on Windows.

For example:

```ini
OPERA = (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = `Host Name / IP Address`)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = OPERA)
  )
)
```

> [!IMPORTANT]
> If using Oracle Database 11g, additional configuration is required to change the TNS authentication protocol. Follow the steps to modify both client and server settings to avoid the error: [ORA-28040: The database does not accept your client's authentication protocol; login denied.](https://docs.oracle.com/en/error-help/db/ora-28040/?r=23ai)

Add the following lines to `sqlnet.ora` for your client pc:

```ini
SQLNET.ALLOWED_LOGON_VERSION_CLIENT = 8
SQLNET.ALLOWED_LOGON_VERSION_SERVER = 8
SQLNET.AUTHENTICATION_SERVICES=(NONE)
```

for tje database server `sqlnet.ora` add the following line (files are typically located in `D:\ORACLE\11204\NETWORK\ADMIN` on Windows).

```ini
SQLNET.AUTHENTICATION_SERVICES=(NONE)
```

---

### Running Queries

1. Go to the Oracle Explorer and connect to the database.
2. Right-click the database and select "Set as Default Connection". The default database connection will have an (\*) flag.
3. To run a query or script, open a new SQL file or create one in your project explorer.
4. Write your query, such as:

   ```sql
   SELECT * FROM resort;
   ```

5. Click the "Execute SQL" button (play icon) to run the query. Results can be exported to `CSV` or `JSON`.

---

### Debugging

The extension includes an inbuilt debugger similar to SQL Developer but with enhanced features from VS Code.
https://medium.com/oracledevs/debugging-pl-sql-with-visual-studio-code-and-more-45631f3952cf


#### Debugging Methods

1. **Simple Debug:** Right-click any function or procedure and select "Debug". Input the required values when prompted.
2. **External Remote Debugger:** Allows you to write and execute custom scripts.

> [!NOTE]
> to resolve the ORA-01031: insufficient privileges, Client PC need to be granted network access through ACL protocols,
> Without this, the database will not be able to remote debug using `SYS.DBMS_DEBUG_JDWP`.

> you might come across following message

```ini
ORA-01031: insufficient privileges
ORA-06512: at "SYS.DBMS_DEBUG_JDWP", line 68
ORA-06512: at line 1
https://docs.oracle.com/error-help/db/ora-01031/
```

For the default database connection, set your PC's IP address under `PL/SQL Debugger Settings` in the Oracle Explorer.

For Oracle Database 12c and above:

```sql
-- Connect as SYSDBA
CONNECT sys/<password>@<datasource> AS sysdba;

-- Grant debug privileges to user
GRANT DEBUG CONNECT SESSION TO <username>; -- usually OPERA
GRANT DEBUG ANY PROCEDURE TO <username>; -- usually OPERA

-- Set up ACL
BEGIN
  DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
    HOST => '<hostname/ip address>', -- your IP address or * to allow all
    LOWER_PORT => <starting port number>, -- from VS Code or NULL for all ports
    UPPER_PORT => <ending port number>, -- from VS Code or NULL for all ports
    ACE => XS$ACE_TYPE(PRIVILEGE_LIST => XS$NAME_LIST('jdwp'),
      PRINCIPAL_NAME => '<username>', -- usually OPERA
      PRINCIPAL_TYPE => XS_ACL.PTYPE_DB)
  );
END;
/
```

For Oracle Database 11g, refer to [this solution](https://stackoverflow.com/questions/65541172/network-access-denied-at-sys-dbms-debug-jdwp) for detailed ACL setup.

```sql
-- Connect as SYSDBA
CONNECT sys/<password>@<datasource> AS sysdba;

-- Grant debug privileges to user
GRANT DEBUG CONNECT SESSION TO <username>; -- usually OPERA
GRANT DEBUG ANY PROCEDURE TO <username>; -- usually OPERA

-- Set up ACL

BEGIN
    DBMS_NETWORK_ACL_ADMIN.CREATE_ACL (
        ACL => 'vscode.xml',
        DESCRIPTION => 'TCP, SMTP, MAIL, HTTP Access',
        PRINCIPAL => 'OPERA',
        IS_GRANT => TRUE,
        PRIVILEGE => 'connect',
        START_DATE => NULL,
        END_DATE => NULL
    );
END;
/

BEGIN
    DBMS_NETWORK_ACL_ADMIN.ASSIGN_ACL (
        ACL => 'vscode.xml',
        HOST => '*',
        LOWER_PORT => NULL,
        UPPER_PORT => NULL
    );
END;
/

BEGIN

    -- OPERA
    DBMS_NETWORK_ACL_ADMIN.ADD_PRIVILEGE (
        ACL => 'vscode.xml',
        PRINCIPAL => 'OPERA',
        IS_GRANT => TRUE,
        PRIVILEGE => 'connect',
        START_DATE => NULL,
        END_DATE => NULL
    );
    DBMS_NETWORK_ACL_ADMIN.ADD_PRIVILEGE (
        ACL => 'vscode.xml',
        PRINCIPAL => 'OPERA',
        IS_GRANT => TRUE,
        PRIVILEGE => 'resolve',
        START_DATE => NULL,
        END_DATE => NULL
    );

END;
/
```

For more information, refer to the [Executing SQL and PL/SQL Documentation](https://docs.oracle.com/en/database/oracle/developer-tools-for-vscode/getting-started/executing-sql-and-pl-sql.html).
