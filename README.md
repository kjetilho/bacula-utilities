# bacula-utilities

This repo contains two utilities, mostly used with Bareos, but should still work with Bacula.  They look directly in the database, so work quickly, at the expense of potential breakage when the schema is updated in the next version.  Please file bugs or submit PRs if you find any issues.

# Setup

The scripts use a simple module called `Bacula::DatabaseConfig`.  The scripts will find it if you install it in the same directory as the scripts themselves, so you can run them directly without any installation.

The module will look for (in order):

* `${HOME}/.bacula-database.cf`
* `/etc/bareos/bacula-database.cf`
* `/etc/bacula/bacula-database.cf`

The file should look something like:

```
database-host bareos.example.com
database-name bareos
database-user bareos_ro
database-password s3kr3t
```

Without such a file, these values must be passed as arguments to the scripts instead, ie., `--database-host bareos.example.com` and so on.

# bacula-du

`bacula-du` works like `du`, except it gets its data from a backup job.  It can also emit an MD5SUM file which can be used to verify that no files have changed since the backup.


```
Usage: bacula-du [OPTIONS] -j JOBID [FILES]
       bacula-du --md5sum -j JOBID [FILES]
Summarize disk usage of directories included in the backup JOBID

Alternate mode:
  --md5sum              output list of all files in job in md5sum format

Options are:
  -a, --all             write counts for all files, not just directories
                        (will skip zero-sized files, see --threshold)
  --apparent-size       use size in octets rather than number of blocks
  -B, --block-size=SIZE report SIZE-byte blocks (default 1Ki)
  -b, --bytes           like --block-size=1 --apparent-size
  -k                    like --block-size=1Ki
  -m                    like --block-size=1Mi
  -i, --inodes          report number of files, not size
  --count-deleted       include deleted files (Accurate mode information)
                        Only relevant for --inodes.
  -S, --separate-dirs   do not include size of subdirectories
  -t, --threshold=SIZE  skip output for files or directories with usage
                        below SIZE.  default is 1 octet.
  -L, --largest=NUM     only print NUM largest directories/files
  -q, --quiet           be quiet (do not print database progress)
  --format FMT          change output format from "%9s %n"
  --stat                multiline output format similar to stat(1), implies -a

SIZE may be (or may be an integer optionally followed by) one of following:
k (1000), Ki (1024), M (1000*1000), Mi (1024*1024), G, Gi, T, Ti, P, Pi.

FMT supports the format sequences from stat(1) as much as possible.
Extensions: 
  %S is file size in units of block-size (default KiB).
  %I is fileindex (Bacula internal information)

If a filename in FILES ends with /, recursion will *not* be done if
that directory is present in the backup (to enable fast --stat).
```

# bacula-logs

`bacula-logs` by default emits all logs from the last backup run (by default starting at 23:00), but has many options to filter output.

```
Usage: bacula-logs [OPTIONS] [JOB ...]

Print logs for JOBS, or print logs for all jobs the last N days (default 1)
JOB can be job id or a job name (optionally with % wildcard).

Options are:

  --errors-only, -e      only print jobs which failed
  --warnings, -w         only print jobs which are OK with warnings or failed
  --ok-only, -O          only print jobs which are OK or OK with warnings
  --running, -r          include still running jobs
  --cancels, -C          include cancelled jobs
  --level-base, -B       only print jobs of level Base
  --level-full, -F       only print jobs of level Full
  --level-differen, -D   only print jobs of level Differential
  --level-incremen, -I   only print jobs of level Incremental
  --schedule-start HH:MM when day starts (default 23:00)
  --days N               how many whole days of logs (default 1)
  --hours N              how many hours of logs
  --backup-exists, -E    only print jobs where backup data exist
                         (will also skip fresh but empty backups)
  --year, -Y             shortcut for --days 365
  --last-run-only, -l    only print the last instance for each job
                         (with --ok-only, print last successful instance)
  --pattern, -p PATTERN  only print jobs with a log matching (Perl) PATTERN [*]
    --ignore-case, -i    case-insensitive pattern search
    --match-only, m      only print matching log entries
    --client-log, -c     only match against log entries from client
                         (without --pattern: only print log entries from client)
  --exclude-jobs PATTERN do not include job with names matching PATTERN [*]
  --status-only, -s      do not print logs
  --extended-info, -X    add field (files, bytes, duration) to status line
  --simple-status, -S    as --status-only, but without decoration of output

[*] may be given multiple times
```

Example:

`bacula-logs -sXOY JOB` will print eXtended status for all jobs within the last Year with Ok status with jobs matching JOB.
