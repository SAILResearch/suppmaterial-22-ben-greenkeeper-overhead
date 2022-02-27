# Greenkeeper Overhead - Supplementary Material

## Getting Started & Greenkeeper In-Range Breaking Build Issue Report (GKIR) Parsers

If you are going to be starting from scratch, you can use the [GitHub Search API](https://docs.github.com/en/rest/reference/search) to find any GKIRs on GitHub by searching for issues with the prefix "An in-range update of" and have been created by the `greenkeeper[bot]` entity. From there, you will have a dataset of GKIRs, as well as a list of client projects on GitHub that have integrated with Greenkeeper. You can then use the [GitHub API](https://docs.github.com/en/rest) to retrieve all other information you need for these client repos (e.g., other issue reports, issuse comments, commits, etc.).

The `gkir_parsers.py` file contains the parser implementations that were used to parse and extract the necesary data from each GKIR. The format that Greenkeeper used to create new GKIRs has changed over the years while Greenkeeper was active, leading to inconsistencies betweeen GKIRs. Additionally, Greenkeeper will somtimes bundle dependency updates for the same monorepo (as explained [here](https://greenkeeper.io/docs.html#monorepo)), so it is necesary to extract information for each dependency being updated in these grouped GKIRs. Therefore different parser implementations are necesary to handle these different formats.

To parse the GKIRs, simply instantiate the `IssueBodyParserImpl` class and call the `parse(body)` method with the body of the GKIR as a parameter. This function will attempt to parse and extract the necesary dependency information using different subclasses of the `IssueBodyParser` parent class. Each subclass handles a different GKIR format. If a subclass failes to parse a GKIR, it simply throws an error, and the next subclass is tried. This process continues until either the GKIR is successfully parsed, or all `IssueBodyParser` subclasses have failed.

The `IssueBodyParserImpl.parse(body)` method returns a list of extracted dependency update records, which consist of the following:

```python
{
    "issue_dependency_name": "@babel/parser", # The name of the provider package.
    "issue_dependency_type": "devDependency", # The type of dependency the provider is in the client project.
    "issue_dependency_actual_version": "7.2.3", # The current version of the provider used in the client project.
    "issue_dependency_next_version": "7.2.4", # The newly released version of the provider that caused the GKIR to be created.
    "issue_dependency_bundle_name": None, # If the GKIR has bundled a group of dependency updates, the name of the bundle.
    "issue_body_parser": "NewDepVersionIssueBodyParser" # The IssueBodyParser used to parse the GKIR.
}
```

If the GKIR is not a bundled GKIR, the list will only have a single record (i.e., only bundled GKIRs will return a list of more than 1 extracted dependency update records).

## Datasets

### `gkirs.csv` - All GKIRs.

|   col_index | attribute                 | type     | description                                                                                            |
|------------:|:--------------------------|:---------|:-------------------------------------------------------------------------------------------------------|
|           0 | id                        | int64    | Issue ID                                                                                               |
|           1 | number                    | int64    | Issue number in project.                                                                               |
|           2 | url                       | str      | GitHub API url of the issue.                                                                           |
|           3 | title                     | str      | Issue title.                                                                                           |
|           4 | state                     | category | Issue state (open, closed, etc.).                                                                      |
|           5 | is_locked                 | bool     | Whether the issue has been locked.                                                                     |
|           6 | created_at                | str      | When the issue was created.                                                                            |
|           7 | updated_at                | str      | When the issue was last updated.                                                                       |
|           8 | closed_at                 | str      | When the issue was closed (if applicable).                                                             |
|           9 | user_login                | category | Username of the entity that created the issue.                                                         |
|          10 | labels                    | str      | Any labels the issue has.                                                                              |
|          11 | num_comments              | int64    | The number of comments on the issue.                                                                   |
|          12 | events_url                | str      | Url for related events.                                                                                |
|          13 | dependency_name           | category | The name of the dependency that caused Greenkeeper to open the issue.                                  |
|          14 | dependency_type           | category | The type of the dependency that caused Greenkeeper to open the issue.                                  |
|          15 | dependency_actual_version | str      | The version of the dependency used by the client project.                                              |
|          16 | dependency_next_version   | str      | The version of the dependency that was just released and caused Greenkeeper to open the issue.         |
|          17 | dependency_bundle_name    | category | The bundle type name of the issue (if applicable).                                                     |
|          18 | body_parser               | category | The parser type that was used to parse the issue (if applicable).                                      |
|          19 | repo_url                  | str      | The url of the client repo.                                                                            |
|          20 | update_type               | category | The update type (major, minor, or patch) between dependency_actual_version and dependency_next_version |
|          21 | repo_name                 | category | The full name of the parent repo.                                                                      |
|          22 | html_url                  | str      | GitHub url of the issue.                                                                               |
|          23 | body                      | str      | The body of the issue.                                                                                 |

---

### `gkir_comments.csv` - All comments created on GKIRs.
|   col_index | attribute                  | type     | description                                                 |
|------------:|:---------------------------|:---------|:------------------------------------------------------------|
|           0 | comment_issue_url          | str      | The GitHub API url of the issue the comment was created on. |
|           1 | comment_issue_id           | int64    | The ID of the issue the comment was created on.             |
|           2 | comment_id                 | int64    | The comment ID.                                             |
|           3 | comment_url                | str      | The GitHub API url of the comment.                          |
|           4 | comment_created_at         | str      | When the comment was created.                               |
|           5 | comment_updated_at         | str      | When the comment was last updated.                          |
|           6 | comment_body               | str      | The body of the comment.                                    |
|           7 | comment_author_association | category | The comment authors association to the parent repo.         |
|           8 | comment_user_id            | int64    | The ID of the entity who created the comment.               |
|           9 | comment_user_login         | category | The username of the entity who created the comment.         |
|          10 | comment_user_type          | category | The type of the entity who created the comment.             |

---

### `gkir_commits.csv` - All commits referenced by GKIRs.
|   col_index | attribute       | type     | description                                                                                               |
|------------:|:----------------|:---------|:----------------------------------------------------------------------------------------------------------|
|           0 | commit_sha      | str      | The commit sha - note that this is not unique in the dataset, as a commit can have multiple file changes. |
|           1 | issue_id        | int64    | The ID of the issue that references the commit.                                                           |
|           2 | repo_name       | category | The full name of the repo the commit belongs to.                                                          |
|           3 | url             | str      | The GitHub API url of the commit.                                                                         |
|           4 | html_url        | str      | The GitHub url of the commit.                                                                             |
|           5 | message         | str      | The commit message                                                                                        |
|           6 | author_login    | category | The username of the author of the commit.                                                                 |
|           7 | author_type     | category | The user type of the author of the commit.                                                                |
|           8 | committer_login | category | The username of the committer of the commit.                                                              |
|           9 | committer_type  | category | The user type of the committer of the commit.                                                             |
|          10 | stats_total     | int64    | The total additions and deletions the commit consists of.                                                 |
|          11 | stats_additions | int64    | The additions the commit consists of.                                                                     |
|          12 | stats_deletions | int64    | The deletions the commit consists of.                                                                     |
|          13 | file_name       | str      | The file name the commit affects.                                                                         |
|          14 | file_status     | category | The type of modification made to the file.                                                                |
|          15 | file_additions  | int64    | The additions of the file change.                                                                         |
|          16 | file_deletions  | int64    | The deletions of the file change.                                                                         |
|          17 | file_changes    | int64    | The total additions and deletions of the file change.                                                     |
|          18 | file_patch      | str      | The raw file patch from the commit.                                                                       |

---

### `non_gkirs.csv` - All non-GKIRs from repos that have integrated with Greenkeeper
|   col_index | attribute       | type     | description                                              |
|------------:|:----------------|:---------|:---------------------------------------------------------|
|           0 | id              | float64  | The issue ID.                                            |
|           1 | repo_name       | category | The full name of the repo the issue was created in.      |
|           2 | url             | object   | The GitHub API url of the issue.                         |
|           3 | repository_url  | str      | The GitHub API url of the repo the issue was created in. |
|           4 | comments_url    | str      | The GitHub API url of the comments for the issue.        |
|           5 | events_url      | str      | The GitHub API url of the events for the issue.          |
|           6 | html_url        | str      | The GitHub HTML url of the issue.                        |
|           7 | number          | float64  | The issue number of the parent repo.                     |
|           8 | title           | str      | The issue title.                                         |
|           9 | user_id         | float64  | The ID of the entity that created the issue.             |
|          10 | user_login      | category | The username of the entity that created the issue.       |
|          11 | user_type       | category | The type of the entity that created the issue.           |
|          12 | state           | category | The current state of the issue.                          |
|          13 | locked          | bool     | Whether the issue has been locked.                       |
|          14 | comments        | float64  | The number of comments on the issue                      |
|          15 | created_at      | str      | When the issue was created.                              |
|          16 | updated_at      | str      | When the issue was last updated.                         |
|          17 | closed_at       | str      | When the issue was closed (if applicable).               |
|          18 | body            | object   | The issue body.                                          |
|          19 | is_pull_request | bool     | Whether the issue is a pull request.                     |

---

### `non_gkir_comments.csv` - All comments created on non-GKIRs.
|   col_index | attribute   | type     | description                                                 |
|------------:|:------------|:---------|:------------------------------------------------------------|
|           0 | id          | int64    | The comment ID.                                             |
|           1 | issue_id    | int64    | The ID of the issue the comment was created on.             |
|           2 | repo_name   | category | The full name of the repo the comment was created in.       |
|           3 | url         | str      | The GitHub API url of the comment.                          |
|           4 | issue_url   | str      | The GitHub API url of the issue the comment was created on. |
|           5 | user_id     | int64    | The ID of the entity who created the comment.               |
|           6 | user_login  | category | The username of the entity who created the comment.         |
|           7 | user_type   | category | The type of the entity who created the comment.             |
|           8 | created_at  | str      | When the comment was created.                               |
|           9 | updated_at  | str      | When the comment was last updated.                          |
|          10 | body        | str      | The body of the comment.                                    |

---

### `non_gkir_commits.csv` - All commits referenced by non-GKIRs.
|   col_index | attribute       | type     | description                                                                                               |
|------------:|:----------------|:---------|:----------------------------------------------------------------------------------------------------------|
|           0 | commit_sha      | str      | The commit sha - note that this is not unique in the dataset, as a commit can have multiple file changes. |
|           1 | issue_id        | float64  | The ID of the issue that references the commit.                                                           |
|           2 | repo_name       | category | The full name of the repo the commit belongs to.                                                          |
|           3 | url             | str      | The GitHub API url of the commit.                                                                         |
|           4 | html_url        | str      | The GitHub url of the commit.                                                                             |
|           5 | message         | str      | The commit message                                                                                        |
|           6 | author_login    | category | The username of the author of the commit.                                                                 |
|           7 | author_type     | category | The user type of the author of the commit.                                                                |
|           8 | committer_login | category | The username of the committer of the commit.                                                              |
|           9 | committer_type  | category | The user type of the committer of the commit.                                                             |
|          10 | stats_total     | int64    | The total additions and deletions the commit consists of.                                                 |
|          11 | stats_additions | int64    | The additions the commit consists of.                                                                     |
|          12 | stats_deletions | int64    | The deletions the commit consists of.                                                                     |
|          13 | file_name       | str      | The file name the commit affects.                                                                         |
|          14 | file_status     | category | The type of modification made to the file.                                                                |
|          15 | file_additions  | int64    | The additions of the file change.                                                                         |
|          16 | file_deletions  | int64    | The deletions of the file change.                                                                         |
|          17 | file_changes    | int64    | The total additions and deletions of the file change.                                                     |
|          18 | file_patch      | str      | The raw file patch from the commit.                                                                       |

---

### `non_gkir_commit_event_relationships.csv` - 
|   col_index | attribute   | type     | description                       |
|------------:|:------------|:---------|:----------------------------------|
|           0 | event_id    | int64    | The associated event ID.          |
|           1 | issue_id    | int64    | The associated issue ID.          |
|           2 | repo_name   | category | The name of the parent repo.      |
|           3 | event_type  | category | The event type.                   |
|           4 | commit_id   | category | The referenced commit sha.        |
|           5 | commit_url  | object   | The GitHub API url of the commit. |

---

### `repos_info.csv` - Information on repos that have integrated with Greenkeeper.

|   col_index | attribute                 | type     | description                                                                        |
|------------:|:--------------------------|:---------|:-----------------------------------------------------------------------------------|
|           0 | repo_name                 | category | The full name of the repo                                                          |
|           1 | is_fork                   | bool     | Whether the repo is a fork.                                                        |
|           2 | size                      | int64    | The size of the repo.                                                              |
|           3 | stargazers_count          | int64    | The number of stars the repo has.                                                  |
|           4 | watchers_count            | int64    | The number of watchers the repo has.                                               |
|           5 | language                  | category | The primary language of the repo.                                                  |
|           6 | package_name              | category | The package name on npm.                                                           |
|           7 | use_repo_name             | bool     | Whether the repo name is different from the name in the project package.json file. |
|           8 | on_libraries_io_npm       | bool     | Whether the package was found on libraries.io.                                     |
|           9 | npm_dependent_repos_count | float64  | The number of dependents on the package - from npm.                                |
|          10 | npm_dependents_count      | float64  | The number of dependents on the package - from npm.                                |
|          11 | npm_forks                 | float64  | The number of forks of the package - from npm.                                     |
|          12 | npm_language              | category | The primary language of the package - from npm.                                    |
|          13 | npm_rank                  | float64  | The rank of the package - from npm.                                                |
|          14 | npm_stars                 | float64  | The number of stars the package has - from npm                                     |
---