# TimescaleDB Changelog

## 0.6.0 (2017-10-09)

**High-level changes**

* Fix bugs where hypertable-specific handlers were affecting normal Postgres tables.
* Make it so that all TimescaleDB commands can run as a normal user rather than a superuser.
* Updates to the code to make the extension compileable on Windows; future changes will add steps to properly build.
* Move `time_bucket` functions out of `public` schema (put in schema where extension is).
* Several other bugs fixes.

**Notable commits**
* [430ed8a] Fix bug with collected commands in index statement.
* [614c2b7] Fix permissions bugs and run tests as normal user
* [ce12104] Fix "ON CONFLICT ON CONSTRAINT" on plain PostgreSQL tables
* [4c451e0] Fix rename and reindex bugs when objects are not relations
* [c3ebc67] Fix permission problems with dropping hypertables and chunks
* [040e815] Remove truncate and hypertable metadata triggers
* [5c26328] Fix INSERT on hypertables using sub-selects with aggregates
* [b57e2bf] Prepare C code for compiling on Windows
* [a2bad2b] Fix constraint validation on regular tables
* [fb5717f] Remove explicit schema for time_bucket
* [04d01ce] Split DDL processing into start and end hooks

**Thanks**
* @oldgreen for reporting `time_bucket` being incorrectly put in the `public` schema and pointing out permission problems
* @qlandman for reporting a bug with INSERT using sub-selects with aggregates

## 0.5.0 (2017-09-20)

**High-level changes**
* Improved support for primary-key, foreign-key, unique, and exclusion constraints.
* New histogram function added for getting the frequency of a column's values.
* Add support for using `DATE` as partition column.
* `chunk_time_interval` now supports `INTERVAL` data types
* Block several unsupported and/or dangerous operations on hypertables and chunks, including dropping or otherwise altering a chunk directly.
* Several bug fixes throughout the code.

**Notable commits**
* [afcb0b1] Fix NULL handling in first/last functions.
* [d53c705] Add script to dump meta data that can be useful for debugging.
* [aa904fa] Block adding constraints without a constraint name
* [a13039f] Fix dump and restore for tables with triggers and constraints
* [8cf8d3c] Improve the size utils functions.
* [2767548] Block adding constraints using an existing index
* [5cee104] Allow chunk_time_interval to be specified as an INTERVAL type
* [6232f98] Add histogram function.
* [2380033] Block ALTER TABLE and handle DROP TABLE on chunks
* [72d6681] Move security checks for ALTER TABLE ALTER COLUMN to C
* [19d3d89] Handle changing the type of dimension columns correctly.
* [17c4ba9] Handle ALTER TABLE rename column
* [66932cf] Forbid relocating extension after install.
* [d2561cc] Add ability to partition by a date type
* [48e0a61] Remove triggers from chunk and chunk_constraint
* [4dcbe61] Add support for hypertable constraints

**Thanks**
* @raycheung for reporting a segfault in `first`/`last`
* @meotimdihia, @noyez, and @andrew-blake for reporting issues with `UNQIUE` and other types of constraints


## 0.4.2 (2017-09-06)

**High-level changes**
* Provide scripts for backing up and restoring single hypertables

**Notable commits**
* [683c078] Add backup/restore scripts for single hypertables

## 0.4.1 (2017-09-04)

**High-level changes**
* Bug fix for a segmentation fault in the planner
* Shortcut when constraint-aware append excludes all chunks
* Fix edge case with negative timestamps when points fell right on the boundary
* Fix behavior of `time_bucket` for `DATE` types by not converting to `TIMESTAMPTZ`
* Make the output of `chunk_relation_size` consistent

**Notable commits**
* [50c8c4c] Fix possible segfault in planner
* [e49e45c] Fix failure when constraint-aware append excludes all chunks
* [c3b6fb9] Fix bug with negative dimension values
* [3c69e4f] Fix semantics of time_bucket on DATE input
* [0137c92] Fix output order of chunk dimensions and ranges in chunk_relation_size.
* [645b530] Convert inserted tuples to the chunk's rowtype

**Thanks**
* @yadid for reporting a segfault (fixed in 50c8c4c)
* @ryan-shaw for reporting tuples not being correctly converted to a chunk's rowtype (fixed in 645b530)

## 0.4.0 (2017-08-21)

**High-level changes**
* Exclude chunks when constraints can be constifyed even if they are
considered mutable like `NOW()`.
* Support for negative values in the dimension range which allows for pre-1970 dates.
* Improve handling of default chunk times for integral date times by forcing it to be explicit rather than guessing the units of the time.
* Improve memory usage for long-running `COPY` operations (previously it would grow unbounded).
* `VACUUM` and `REINDEX` on hypertables now recurse down to chunks.

**Notable commits**
* [139fe34] Implement constraint-aware appends to exclude chunks at execution time
* [2a51cf0] Add support for negative values in dimension range
* [f2d5c3f] Error if add_dimension given both partition number and interval length
* [f3df02d] Improve handling of non-TIMESTAMP/TZ timestamps
* [6a5a7eb] Reduce memory usage on long-running COPY operations
* [953346c] Make VACUUM and REINDEX recurse to chunks
* [55bfdf7] Release all cache pins when a transaction ends

## 0.3.0 (2017-07-31)

**High-level changes**
* "Upserts" are now supported via normal `ON CONFLICT DO UPDATE`/`ON CONFLICT DO NOTHING` syntax. However, `ON CONFLICT ON CONSTRAINT` is not yet supported.
* Improved support for user-defined triggers on hypertables. Now handles both INSERT BEFORE and INSERT AFTER triggers, and triggers can be named arbitrarily (before, a \_0\_ prefix was required to ensure correct execution priority).
* `TRUNCATE` on a hypertable now deletes empty chunks.

**Notable commits**
* [23f9d3c] Add support for upserts (`ON CONFLICT DO UPDATE`)
* [1f3dcd8] Make `INSERT`s use a custom plan instead of triggers
* [f23bf58] Remove empty chunks on `TRUNCATE` hypertable.

## 0.2.0 (2017-07-12)

**High-level changes**
* Users can now define their own triggers on hypertables (except for `INSERT AFTER`)
* Hypertables can now be renamed or moved to a different schema
* Utility functions added so you can examine the size hypertables, chunks, and indices

**Notable commits**
* [83c75fd] Add support for triggers on hypertables for all triggers except `INSERT AFTER`
* [e0eeeb9] Add hypertable, chunk, and indexes size utils functions.
* [4d2a65d] Add infrastructure to build update script files.
* [a5725d9] Add support to rename and change schema on hypertable.
* [142f58c] Cleanup planner and process utility hooks

## 0.1.0 (2017-06-28)

**IMPORTANT NOTE**

Starting with this release, TimescaleDB will now
support upgrading between extension versions using the typical
`ALTER EXTENSION` command, unless otherwise noted in future release notes. This
important step should make it easier to test TimescaleDB and be able
to get the latest benefits from new versions of TimescaleDB. If you
were previously using a version with the `-beta` tag, you will need
to `DROP` any databases currently using TimescaleDB and re-create them
in order to upgrade to this new version. To backup and migrate data,
use `pg_dump` to save the table schemas and `COPY` to write hypertable
data to CSV for re-importing after upgrading is complete. We describe
a similar process on [our docs](http://docs.timescale.com/getting-started/setup/migrate-from-postgresql#different-db).

**High-level changes**
* More refactoring to stabilize and cleanup the code base for supporting upgrades (see above note)
* Correct handling of ownership and permission propagation for hypertables
* Multiple bug fixes

**Notable commits**
* [696cc4c] Provide API for adding hypertable dimensions
* [97681c2] Fixes permission handling
* [aca7f32] Fix extension drop handling
* [9b8a447] Limit the SubspaceStore size; Add documentation.
* [14ac892] Fix possible segfault
* [0f4169c] Fix check constraint on dimension table
* [71c5e78] Fix and refactor tablespace support
* [5452dc5] Fix partiton functions; bug fixes (including memory)
* [e75cd7e] Finer grained memory management
* [3c460f0] Fix partitioning, memory, and tests
* [fe51d8d] Add native scan for the chunk table
* [fc68baa] Separate out subspace_store and add it to the hypertable object as well
* [c8124b8] Use hypercube instead of dimension slice list
* [f5d7786] Change the semantics of range_end to be exclusive
* [700c9c8] Refactor insert path in C.
* [0584c47] Created chunk_get_or_create in sql with an SPI connector in C
* [7b8de0c] Refactor catalog for new schema and add native data types
* [d3bdcba] Start refactoring to support any number of partitioning dimensions

## 0.0.12-beta (2017-06-21)

**High-level changes**
* A major cleanup and refactoring was done to remove legacy code and
currently unused code paths. This change is **backwards incompatible**
and will require a database to be re-initialized and data re-imported.
This refactoring will allow us to provide upgrade paths starting with
the next release.
* `COPY` and `INSERT` commands now return the correct number of rows
* Default indexes no longer duplicate existing indexes
* Cleanup of the Docker image and build process
* Chunks are now time-aligned across partitions

**Notable commits**
* [3192c8a] Remove Dockerfile and docker.mk
* [2a01ebc] Ensure that chunks are aligned.
* [73622bf] Fix default index creation duplication of indexes
* [c8872fe] Fix command-tag for COPY and INSERT
* [bfe58b6] Refactor towards supporting version upgrades
* [db01c84] Make time-bucket function parallel safe
* [18db11c] Fix timestamp test
* [97bbb59] Make constraint exclusion work with non-text partition keys
* [f2b42eb] Fix problems with partitioning logic for padded fields
* [997029a] if_not_exist flag to create_hypertable now works on hypertables with data as well
* [347a8bd] Reference the correct column when scanning partition epochs
* [88a9849] Fix bug with timescaledb.allow_install_without_preload GUC not working

## 0.0.11-beta (2017-05-24)

**High-level changes**
* New `first(value, time)` and `last(value, time)` aggregates
* Remove `setup_timescaledb()` function to streamline setup
* Allow for use cases where restarting the server is not feasible by force loading the library
* Disable time series optimizations on non-hypertables
* Add some default indexes for hypertables if they do not exist
* Add "if not exists" flag for `create_hypertable`
* Several bug fixes and cleanups

**Notable commits**
* [8ccc8cc] Add if_not_exists flag to create_hypertable()
* [2bc60c7] Fix time interval field name in hypertable cache entry
* [4638688] Improve GUC handling
* [cedcafc] Remove setup_timescaledb() and fix pg_dump/pg_restore.
* [34ad9a0] Add error when timescaledb library is not preloaded.
* [fc4ddd6] Fix bug with dropping chunks on tables with indexes
* [32215ff] Add default indexes for hypertables
* [b2900f9] Disable query optimization on regular tables (non-hypertables)
* [f227db4] Fixes command tag return for COPY on hypertables.
* [eb32081] Fix Invalid database ID error
* [662be94] Add the first(value, time),last(value, time) aggregates
* [384a8fb] Add regression tests for deleted unit tests
* [31ee92a] Remove unit tests and sql/setup
* [13d3acb] Fix bug with alter table add/drop column if exists
* [f960c24] Fix bug with querying a row as a composite type

## 0.0.10-beta (2017-05-04)

**High-level changes**
* New `time_bucket` functions for doing roll-ups on varied intervals
* Change default partition function (thanks @robin900)
* Variety of bug fixes

**Notable commits**
* [1c4868d] Add documentation for chunk_time_interval argument
* [55fd2f2] Fixes command tag return for `INSERT`s on hypertables.
* [c3f930f] Add `time_bucket` functions
* [b128ac2] Fix bug with `INSERT INTO...SELECT`
* [e20edf8] Add better error checking for index creation.
* [72f754a] use PostgreSQL's own `hash_any` function as default partfunc (thanks @robin900)
* [39f4c0f] Remove sample data instructions and point to docs site
* [9015314] Revised the `get_general_index_definition` function to handle cases where indexes have definitions other than just `CREATE INDEX` (thanks @bricklen)
