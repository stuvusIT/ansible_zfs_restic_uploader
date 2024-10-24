# zfs_restic_uploader

Ansible role that deploys our `zfs-restic-uploader` script and configures a corresponding systemd service and timer.

## Role Variables

| Name                            | Required/Default   | Description                                                                                                                                                               |
| ------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `zru_access_key_id`             | :heavy_check_mark: | S3 access key for the restic repository                                                                                                                                   |
| `zru_secret_access_key`         | :heavy_check_mark: | S3 secret key for the restic repository                                                                                                                                   |
| `zru_restic_repo_password`      | :heavy_check_mark: | Restic repository password (for encryption at rest)                                                                                                                       |
| `zru_restic_repo_prefix`        | :heavy_check_mark: | The S3 url (possibly including a prefix inside the bucket) used for the restic repo. It is appended with the dataset name.                                                |
| `zru_restic_check`              | `True`             | Whether to run `restic check` after finishing uploading to a Restic repository                                                                                            |
| `zru_schedule`                  | `"*-*-* 4:00:00"`  | Schedule for systemd timer                                                                                                                                                |
| `zru_keep_last_n`               | `0`                | Number of last snapshots to keep. This gets passed to `zfs-restic-uploader`'s `--keep-last-n` flag.                                                                       |
| `zru_keep_weekly_n`             | `0`                | Number of weekly snapshots to keep. This gets passed to `zfs-restic-uploader`'s `--keep-last-n` flag.                                                                     |
| `zru_keep_monthly_n`            | `0`                | Number of monthly snapshots to keep. This gets passed to `zfs-restic-uploader`'s `--keep-last-n` flag.                                                                    |
| `zru_cache_directory`           | `/var/cache`       | Cache directory for Restic. This gets passed via the environment variable `$XDG_CACHE_HOME`.                                                                              |
| `zru_exclude_snapnames_regex`   | `a^`               | Snapshots whose snapname matches this regex are ignored. The default of `a^` is a regex that is impossible to match, so nothing will be ignored.                          |
| `zru_release_holds`             | `[]`               | List of ZFS holds that are released after a snapshot was processed (i.e., successfully uploaded or skipped) except on snapshots excluded by `zru_exclude_snapnames_regex` |
| `zru_zfs_dataset_common_prefix` | `""`               | The prefix which should be removed from each dataset name for use in the restic repo. E.g. `backup01`                                                                     |
| `zru_zfs_datasets`              | `[]`               | Names of the datasets to backup.                                                                                                                                          |

## Retention

The `zru_keep_*` variables configure the desired retention policy analogous to the flags supported by `restic forget`.
However, note that `zfs-restic-uploader` doesn't delete anything.
When a snapshot doesn't meet the retention policy, that only means that it will not be uploaded (and ZFS holds listed in `zru_release_holds` get released).
If they are already uploaded, they will not be deleted from the Restic repository by this tool.

In order to delete snapshots that don't (anymore) meet the retention policy, you need to run `restic forget` on the repository yourself.

When all of the `zru_keep_*` variables are set to `0`, then a special case applies where all (instead of zero) snapshots are uploaded (except of course those snapshots excluded via `zru_exclude_snapnames_regex`).

## License

This work is licensed under the [MIT License](./LICENSE).
