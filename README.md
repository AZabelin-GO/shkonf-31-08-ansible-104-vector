# Vector Role

Simple Ansible role to install Vector (observability data collector) on Debian/Ubuntu systems and configure a demo `demo_logs` source that forwards events to ClickHouse.

## What it does

- Adds the Vector APT repository and GPG key.
- Installs the configured Vector package(s).
- Installs `/etc/vector/vector.yaml` from `templates/vector.yaml.j2` using role variables.
- Ensures the `vector` systemd service is enabled and restarted when the config changes.

## Supported platforms

- Debian / Ubuntu (APT-based systems with systemd)

## Requirements**

- Network access to `{{ apt_vector_repository_url }}` to download packages.
- `systemd` on the target host (role manages the `vector` systemd unit).
- ClickHouse reachable from the host if you intend to use the bundled ClickHouse sink in the template.

## Variables

The following variables are provided by this role (defaults in `defaults/main.yml` and `vars/main.yml`):

- `clickhouse_url` (default: `http://clickhouse:8123`) — ClickHouse HTTP endpoint used by the `clickhouse` sink.
- `clickhouse_db_name` (default: `vector`) — ClickHouse database name.
- `clickhouse_table_name` (default: `logs`) — ClickHouse table name to write events to.
- `clickhouse_user` (default: `vector`) — ClickHouse basic auth user.
- `clickhouse_password` (default: `vector`) — ClickHouse basic auth password.

- `vector_major_version` (default: `0`) — Major version used in the APT component naming.
- `vector_dist_channel` (default: `stable`) — Repository suite/channel (e.g. `stable`).
- `apt_vector_repository_url` (default: `https://apt.vector.dev/`) — APT repo base URL used to add the Vector repository.
- `apt_vector_gpg_key_url` (default: `https://keys.datadoghq.com/DATADOG_APT_KEY_CURRENT.public`) — URL to the GPG key used to sign the Vector packages.
- `apt_vector_packages` (default: `['vector=0.57.0-1']`) — List of packages (and versions) to install. Pin or change the version as needed.

All variables are defined in `defaults/main.yml` and `vars/main.yml`, and can be overridden in your playbook, inventory, or group/host vars.

## Files and templates

- Template: [templates/vector.yaml.j2](templates/vector.yaml.j2) — role provides a ready-to-use Vector config using a `demo_logs` source and a ClickHouse sink.
- Tasks: [tasks/main.yml](tasks/main.yml) — repository setup, package install, config template, and systemd service management.
- Handlers: [handlers/main.yml](handlers/main.yml) — restarts the `vector` service when config changes.

## Dependencies

- None.

## Example Playbook

Overwrite defaults as needed, for example to point at an external ClickHouse instance:

```yaml
- hosts: monitoring
  become: true
  roles:
    - role: vector
      vars:
        clickhouse_url: "http://clickhouse.example.local:8123"
        clickhouse_user: "vector_user"
        clickhouse_password: "s3cret"
```

## Notes and recommendations

- The provided `templates/vector.yaml.j2` uses the `demo_logs` source (useful for testing). Replace or extend sources/sinks for production use.
- The role pins the Vector package in `vars/main.yml` — update the pinned version carefully when upgrading.
- Ensure ClickHouse is reachable and configured with the expected database/table or modify the template to match your ClickHouse schema/setup.

## License

MIT

## Author Information

Role author: AZabelin
