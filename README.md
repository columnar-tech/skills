<!--
Copyright 2026 Columnar Technologies Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Agent Skills

[Agent skills](https://agentskills.io) for [ADBC](https://arrow.apache.org/adbc), [dbc](https://docs.columnar.tech/dbc), [databow](https://docs.columnar.tech/databow), and the [DuckDB ADBC extension](https://duckdb.org/community_extensions/extensions/adbc).

## Installation

There are multiple automated ways to install these skills:

```sh
# GitHub CLI (https://cli.github.com)
$ gh skill install github.com/columnar-tech/skills
# skills package (https://www.skills.sh)
$ npx skills add columnar-tech/skills
```

Note: To update your local versions of these skills once you install them, both `gh skill` and `npx skills` have update subcommands.

If you prefer manual installation, refer to your coding agents' documentation for skill installation.

## Available Skills

- `adbc` — Skill for using [ADBC](https://arrow.apache.org/adbc) (Arrow Database Connectivity)
- `dbc` — Skill for using [dbc](https://docs.columnar.tech/dbc) (the CLI for installing and managing ADBC drivers)
- `databow` — Skill for using [databow](https://docs.columnar.tech/databow) (the CLI for querying databases)
- `duckdb-adbc` — Skill for using the [DuckDB ADBC extension](https://duckdb.org/community_extensions/extensions/adbc) (the DuckDB extension for querying databases with ADBC)

## Contributing

If you'd like to contribute, please file an [issue](https://github.com/columnar-tech/skills/issues). Good ways to contribute are:

- Reporting issues installing or using the skills in this repo
- Ideas for new skills
