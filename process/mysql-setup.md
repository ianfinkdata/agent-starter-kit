# One-time MySQL Setup for Claude Code

## 1. Credentials file (default location: ~/.my.cnf)
MySQL reads option files in order: /etc/my.cnf → /etc/mysql/my.cnf → ~/.my.cnf.
Your personal credentials belong in ~/.my.cnf:

    [client]
    user = claude_ro
    password = <password>
    host = 127.0.0.1

Then lock it down:

    chmod 600 ~/.my.cnf

Verify it works with no flags:

    mysql -e "SELECT 1;"

## 2. Create a read-only user for agents (run as root/admin)

    CREATE USER 'claude_ro'@'localhost' IDENTIFIED BY '<password>';
    GRANT SELECT, SHOW VIEW ON yourdb.* TO 'claude_ro'@'localhost';
    FLUSH PRIVILEGES;

Agents physically cannot write, drop, or alter — your QA gate stays real.

## 3. How agents connect
No MCP server needed. Claude Code's Bash tool runs the mysql CLI, which
auto-loads ~/.my.cnf. Agents run:

    mysql yourdb --batch -e "SELECT ... LIMIT 100;"

The contract (CLAUDE.md §Database access) forbids agents from reading or
printing ~/.my.cnf and requires escalation for any write operation.

## 4. Optional: separate profile for agents
If your personal ~/.my.cnf uses an admin user, add a suffixed group so
agents use the read-only account:

    [client_claude]
    user = claude_ro
    password = <password>
    host = 127.0.0.1

Agents then connect with:

    mysql --defaults-group-suffix=_claude yourdb -e "..."

## 5. Optional: MCP route (only if you want structured schema tools)
A community MySQL MCP server can be added as a local stdio server:

    claude mcp add --transport stdio mysql -- npx -y <mysql-mcp-package>

The CLI route above is simpler and keeps credentials out of all configs,
so start there.
