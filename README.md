# Gitlab Migration

Script to migrate Gitlab groups and their projects from one Gitlab instance to
another.

Can be used to migrate groups/repos within the same instance, by providing the
same path for SOURCE_GITLAB and TARGET_GITLAB, and the same PAT on both lines in
`.secrets`

Non-extensive list of features exported:

- Tags
- Commits
- Open MRs
- Issues
- CI/CD Variables (group/project) _Optional_
- Export all projects in a group
- If you have to re-run due to hitting a rate limit, will skip projects already
  imported into the target area

## Requirements

- [Bash (4.0 or newer)](https://apple.stackexchange.com/questions/193411/update-bash-to-version-4-0-on-osx)
- jq `brew install jq`
- [Grep (3.0 or newer)](https://apple.stackexchange.com/questions/193288/how-to-install-and-use-gnu-grep-in-macos)

## Usage

- Create local file `.secrets` and add
  a [Personal Access Token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)
  for the source Gitlab instance in the first line and a Personal Access Token
  for the target Gitlab instance in the second line. Both tokens need the `api`
  scope. Example:

  ```
  $ cat .secrets
  dskjfdskjfr7987dfkds
  jdskfsdjflk238dg762a
  ```

- Create the target group in the target Gitlab instance. **CAUTION: You should
  always make sure that you set the appropriate visibility level and other
  important security settings on the target group before you start the migration
  so that no sensitive data is exposed by accident!** See details on additional
  protective default settings at [Supported Features](#supported-features).
- Run the migration script:

  ```bash
  $ SOURCE_GITLAB=git.mycompany.com TARGET_GITLAB=gitlab.com ./migrate.sh
  Enter source group path at git.mycompany.com (e.g. project/team):
  Enter target group path at gitlab.com (e.g. mycompany/project/team):
  Archive original projects at git.mycompany.com after migration? (yes/no):
  ...
  ```

## Limitations

- Tested with Gitlab 15.6
- Currently only Community Edition features are supported.
  See [list of supported features](#supported-features) below for details. This
  doesn't limit the script to Community Edition Gitlab instances though. It can
  be used with any of Community, Enterprise or Gitlab.com SaaS instances as
  source and target.
- The target group path already has to exist
- Max. 100 Subgroups, Variables, Hooks, Badges per parent entity will be
  migrated as no API pagination is implemented
- If you hit an error during the migration, you might have to wait several
  minutes before resuming, as Gitlab has some pretty tight rate limiting
  restrictions for project
  exports: [rate limits](https://docs.gitlab.com/ee/user/project/settings/import_export.html#rate-limits)

## Supported Features

- Groups including nested groups.
  See [Group API documentation](https://docs.gitlab.com/ee/api/groups.html#list-groups)
  for supported attributes.
  The following attributes will be overwritten with some restrictive values to
  not expose sensitive data by default. This is especially handy if you're
  moving from a private on-premise instance to Gitlab.com.
  You can always change the settings back after the migration manually or edit
  the script to not override these attributes if you want to keep the original
  settings.
  1. `visibilty=private`
  1. `request_access_enabled=false`
  1. `require_two_factor_authentication=true`
  1. `share_with_group_lock=true`
- Project Export Package.
  See [Gitlab Import/Export documentation](https://docs.gitlab.com/ee/user/project/settings/import_export.html#exported-contents)
  for details
- Optional:
  1. Group CI variables
  1. Group badges
  1. Project CI variables
  1. Project webhooks
  1. Add link to new location in source project description
  1. Archive source projects after migration
  1. Dry Run - prints out to console the list of projects that will be
     migrated
  1. Single project export - enter the name of a singular project to export
     from
     a group (useful for testing full functionality on a group with many
     projects)

## Contributions

- Cut a branch
- Pre-commit
  1. Install pre-commit (v2.18 or above) - using brew or pip
  1. Brew install shellcheck,
     [ruby@3.0](https://mac.install.guide/ruby/13.html), semgrep
- Run `pre-commit install --hook-type` to setup pre-commit

## License

Copyright (c) 2020 K - Mail Order GmbH & Co. KG

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the
License at

[Apache license](https://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.

## Legal Notice

GIT is a trademark of Software Freedom Conservancy. Gitlab is a trademark of
Gitlab B.V.
