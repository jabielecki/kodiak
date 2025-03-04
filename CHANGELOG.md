# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## 0.18.0 - 2020-03-04

### Added
- add web dashboard accessible at https://app.kodiakhq.com. This website enables viewing Kodiak activity.
- add `merge.message.include_pull_request_author` configuration option to append pull request author information as a coauthor in the merge commit. (#301)

## 0.17.0 - 2020-01-26

### Added
- add `approve.auto_approve_usernames` to enable Kodiak to auto approve PRs. This option enables bots like Dependabot to automatically merge PRs when the GitHub Branch Protection "Required approving reviews" is configured. See https://kodiakhq.com/docs/recipes#automated-dependency-updates-with-dependabot for an example. (#260)

## 0.16.0 - 2020-01-25

### Fixed
- fixed status event handler triggering reevaluations of all PRs in a repository. Now we only trigger updates for PRs directly related to a status event. (#248)
- replaced inaccurate webhook event schemas with simplified versions to curtail parsing errors. Now we only parse the little information we need from each webhook event. This issue was preventing some webhook events from triggering reevaluations of PRs. (#262, #261)

## 0.15.0 - 2020-01-18

### Fixed
- fixed `merge.delete_branch_on_merge` deleting branches that had open PRs against them. This fix eliminates a confusing bug where it would look like Kodiak closed the dependent PR. What happened was Kodiak deleted a branch on which that PR was dependent, so the PR was forced to be closed by GitHub. (#232)
- fixed bug in webhook event handling where we wouldn't trigger evaluation for PRs when their dependent branch updated. We now use the `push` event to trigger evaluation of PRs that depend on the pushed ref. (#244)

## 0.14.0 - 2020-01-12

### Added
- add support for placing `.kodiak.toml` at `.github/.kodiak.toml`. (#227)


### Changed
- updated warnings to allow commit signature branch protection setting when "merge" is configured as Kodiak's merge method. Kodiak is able to create signatures for merge commits, but not for squash and rebase merge methods (GitHub limitation). (#230)


### Fixed
- add handling to support reviews created by bots. A bot is not compatible with user API endpoints, so when a bot review was added Kodiak would fail when evaluating permissions on the bot. (#231)

## 0.13.0 - 2020-01-06

### Added
- add `update.always` and `update.require_automerge_label` configuration options. When `update.always = true`, Kodiak will update a branch immediately, regardless of failing mergeability requirements (e.g. missing/failing checks, title blacklist regex, blacklist labels). When `update.require_automerge_label = false` with `update.always = true`, Kodiak will update a PR even if missing the automerge label defined in `merge.automerge_label`. (#174, #198, #213)

### Deprecated
- discourage use of `merge.update_branch_immediately` configuration option. This setting will not be removed, but its use is discouraged because it can produce unexpected results. The behavior of `update.always` is easier to understand. (#198)

## 0.12.0 - 2020-01-04

### Changed
- refactored core update/merge eligibility logic. This was a large change and should make future features significantly easier to implement and test. (#207)


### Security
- removed potential Regex Denial of Service (ReDoS) vulnerability from `merge.blacklist_title_regex` by using a regex engine ([rure][rure-python]) that guarantees linear time searching. (#211)

[rure-python]: https://github.com/davidblewett/rure-python

## 0.11.0 - 2019-12-20

### Added
- updating of PRs made from forks. The merges API endpoint Kodiak had been using for updating branches didn't work across forks due to GitHub permissions. A [new API endpoint][update-branch-blog] was released in late May 2019 that avoided any permission issue, but wasn't noticed until 2019-12-12 🤦‍♀️. This change should make Kodiak more useful for public projects. (#104, #202)

[update-branch-blog]: https://developer.github.com/changes/2019-05-29-update-branch-api/

## 0.10.0 - 2019-12-04

### Added
- `GITHUB_PRIVATE_KEY_BASE64` environment variable to support configuring GitHub private key via base64. This is a workaround to support Docker's .env files, which do not allow multi-line or quoted values (#191, #192).
- `merge.do_not_merge` configuration option to support updating PRs without merging them (#187).

### Changed
- deprecate `merge.block_on_reviews_requested`, which is fundamentally broken and cannot be fixed (#180, #182).

### Fixed
- fixed travis-ci check compression to support deprecated travis-ci status check format (#166).

## 0.9.0 - 2019-09-07

### Added
- `merge.prioritize_ready_to_merge` configuration option to immediately merge a PR if it's mergeable instead of placing it in the merge queue. This allows PRs to bypass those waiting to update in the queue if they are mergeable. See the README for more details.

## 0.8.0 - 2019-08-11

### Added
- `merge.update_branch_immediately` configuration option to immediately update a PR when the target is updated instead of waiting until just before the PR is merged. See [README.md#config-with-comments-and-all-options-set](https://github.com/chdsbd/kodiak#config-with-comments-and-all-options-set) for a more detailed explanation of this feature and potential drawbacks (#120)

## 0.7.0 - 2019-08-01

### Fixed
- fixed updating PR accidentally removing it from the merge queue (#148)
- fixed possible race condition when `dont_wait_on_status_checks` was configured that could accidentally remove a PR from the merge queue (#149)

## 0.6.0 - 2019-07-28

### Added
- display requested reviewer names in status messages (#130)
- add warning that forks cannot be updated when PR is from fork (#135)
- add nicer error message for unknown block reason. The previous message erroneously indicated there was a problem with Kodiak (#139)
- add configuration to ignore select pending status checks. This is useful to prevent waiting indefinitely for the [WIP GitHub App](http://github.com/marketplace/wip) Check Run to complete (#141)

### Fixed
- ensure user has write permissions when counting their reviews towards mergeability. We previously checked the wrong field for this information. (#134)

## 0.5.0 - 2019-07-26

### Added
- configuration for redis connection pool size (#57)
- `merge.optimistic_updates` configuration to prioritize updates over status checks (#64)
- Internal rate limiting of API calls to Github (#70)
- Improve status check messages when PR is being merged (#72)
- Add queue position information to status messages (#74)
- Display more information in status checks (#77, #112)
    + has blacklist labels reports the blacklist labels
    + invalid merge method displays the configured merge method and the valid merge methods
    + blocking review shows first blocking user
    + missing required review count shows review count and required review count
    + failing required status checks displays the failing status checks
    + waiting on status checks now reports the status checks we are waiting for
- Display status messaging for missing branch protection (#78)
- Add `merge.message.strip_html_comments` configuration option to strip comments from (#80)
    + This is useful for stripping HTML comments created by PR templates when the `markdown` `body_type` is used.
- Add `require_automerge_label` to configure requirement of `automerge_label` for working on PR (#82)
    + This overrides and disables `notify_on_conflict` (#86)
- Add status message warning for unsupported `requiresCommitSignatures` branch protection configuration (#90, #91)
    + This is a limitation of the Github API as Kodiak is not able to created signed commits when merging a PR.
- Add status message reporting of update branch failures (#94)
- Add "Known issues" section to README (#105, #114)
- Add `'empty'` configuration option for `merge.message.body` to truncate PR body on merge (#111)
- Display configuration parsing errors with details page when kodiak cannot parse a configuration file (#116, #125)


### Fixed
- Support PRs in draft state (#68)
- Fix bug where `require_automerge_label` would trigger kodiak to make an infinite loop of comments (#86)
- Fix poor status message templating on update branch failure case (#110)

## 0.4.0 - 2019-06-16

### Added
- Add support for `include_pr_number` configuration. Enabling `include_pr_number` with a non-default merge message option will append the pr number to the commit message, like the Github UI.
- Make merge body style (plain text, markdown, or HTML) configurable via `merge.message.body_type`.
- Add automatic deletion of branches on merge configurable via `merge.delete_branch_on_merge`.
- Add support for running multiple kodiak instances on the same repo via the `app_id` configuration option.
- Add redis-based persistence. Redis >=5 is now required.
- Add `merge.blacklist_title_regex` configuration to block merging PRs that match configured regex.
- Display kodiak status information in Github CheckRun.
- Add docs for testing kodiak locally.
- Add automerge label removal and PR comment when a merge conflict occurs. This is configurable via `merge.notify_on_conflict`.

### Changed
- move `block_on_reviews_requested` to `merge.block_on_reviews_requested`.
- replace `merge.whitelist` array with singular `merge.automerge_label`.
- rename `merge.blacklist` to `merge.blacklist_labels`.

## 0.3.0 - 2019-05-24

### Added
- Add support for configuring merge messages. Current options are default Github style or using pull request title and body.

### Fixed
- Fix handling of `CHANGE_REQUESTED` reviews. We weren't nullifying `CHANGE_REQUESTED` reviews after the user placed another review.

## 0.2.1 - 2019-05-23

### Fixed
- Fix incorrect calls to sentry client. `send_message` => `capture_message`
- Fix missing handling for Github CheckRuns.

## 0.2.0 - 2019-05-22

### Added
- Add `block_on_reviews_requested` configuration to block merging if there are unanswered review requests.

### Changed
- Update mergeability evaluation to trigger branch update after all other mergeability tests are verified.

## 0.1.0 - 2019-05-22

### Added

- Basic MVP
