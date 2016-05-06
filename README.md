# PGDump to S3
A small utility to dump a database, gzip it, and upload to S3

## Requirements

Set these env-vars for pg_dump (see [PG ENV-vars](http://www.postgresql.org/docs/current/static/libpq-envars.html)):
  * PGHOST
  * PGPORT (unless 5432)
  * PGUSER
  * PGPASSWORD
(and if need be, other PG env vars according to your needs)

and these for AWS CLI (or less if using EC2-roles) (see [AWS CLI ENV-vars](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-environment)):
  * AWS_ACCESS_KEY_ID
  * AWS_SECRET_ACCESS_KEY
  * AWS_DEFAULT_REGION

## Temp-volume

Be sure to bind a temporary volume to `/backuptemp` (or somewhere else, but set env `TEMP_BACKUP_DIR` to that).

## Command
Entrypoint is the script [dump_and_save](dump_and_save).

The script takes these parameters:
    `s3://mybucket/myfolder database_1 database_2`

so you can dump several named databases in one go.
The utility will gzip to a file named as `[database_name].gz` and then upload it to S3. It will replace an already present file.

## pg_dump parameters

Utility will use these parameters for pg_dump :
`pg_dump -Fc --no-acl --no-owner`

explanied as (from `man pg_dump`):
  * `-Fc` = Format custom - Output a custom-format archive suitable for input into pg_restore. Together with the directory output format, this is the most flexible output format in that it allows manual selection and reordering of archived items during restore. This format is also compressed by default.
  * `--no-acl` = Prevent dumping of access privileges (grant/revoke commands).
  * `--no-owner` = Do not output commands to set ownership of objects to match the original database.

## Example

`docker run --rm -it -e PGHOST=xx -e PGUSER=xx -e PGPASSWORD=xx -e AWS_DEFAULT_REGION=eu-west-1 -e AWS_ACCESS_KEY_ID=xx -e AWS_SECRET_ACCESS_KEY=xx -v /backuptemp upptec/pgdump_to_s3 s3://my_bucket/my_folder mydb`

## Howto restore
Note: we use it to restore to a local db, mostly for developing purposes, so `pg_restore`-command below reflects that.

  * After downloading, unpack using `gzip -d mydump.gz`.
  * Then: `pg_restore --verbose --clean --no-acl --no-owner -h localhost -U myuser -d mydb mydump`.

Since this utility uses same name on dump as database, and we use our local username as db-user, this is would be the syntax for database `mydb`, unpacked from file `mydb.gz`:

    pg_restore --verbose --clean --no-acl --no-owner -h localhost -U $USER -d mydb mydb
