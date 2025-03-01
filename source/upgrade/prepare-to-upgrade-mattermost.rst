Prepare to upgrade Mattermost
=============================

|all-plans| |self-hosted|

.. |all-plans| image:: ../images/all-plans-badge.png
  :scale: 30
  :target: https://mattermost.com/pricing
  :alt: Available in Mattermost Free and Starter subscription plans.

.. |self-hosted| image:: ../images/self-hosted-badge.png
  :scale: 30
  :target: https://mattermost.com/deploy
  :alt: Available for Mattermost Self-Hosted deployments.

In most cases, you can `upgrade Mattermost Server <https://docs.mattermost.com/upgrade/upgrading-mattermost-server.html>`__ in a few minutes. However, the upgrade can take longer depending on several factors, including the size and complexity of your installation, and the version that you're upgrading from.

.. contents::
  :backlinks: top
  :local:

Upgrade to Mattermost v7.1
--------------------------

Mattermost v7.1 introduces schema changes in the form of a new column and its index. The following notes our test results for the schema changes:

- MySQL 12M Posts, 2.5M Reactions - ~1min 34s (instance: PC with 8 cores, 16GB RAM)
- PostgreSQL 12M Posts, 2.5M Reactions - ~1min 18s (instance: db.r5.2xlarge)

You can run the following SQL queries before the upgrade that obtains a lock on ``Reactions`` table. Users' reactions posted during this time won't be reflected in the database until the migrations are complete. This is fully backwards-compatible.

.. tabs:: 

  .. tab:: MySQL

    ``ALTER TABLE Reactions ADD COLUMN ChannelId varchar(26) NOT NULL DEFAULT "";``

    ``UPDATE Reactions SET ChannelId = COALESCE((select ChannelId from Posts where Posts.Id = Reactions.PostId), '') WHERE ChannelId="";``

    ``CREATE INDEX idx_reactions_channel_id ON Reactions(ChannelId) LOCK=NONE;``

  .. tab:: PostgreSQL

    ``ALTER TABLE reactions ADD COLUMN IF NOT EXISTS channelid varchar(26) NOT NULL DEFAULT '';``

    ``UPDATE reactions SET channelid = COALESCE((select channelid from posts where posts.id = reactions.postid), '') WHERE channelid='';``

    ``CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_reactions_channel_id on reactions (channelid);``

Upgrade to Mattermost v7.0
--------------------------

Self-hosted Mattermost customers using MySQL databases may notice the migration to release v7.0 taking longer than usual when there are a large number of rows in the ``FileInfo`` table. See the `important upgrade notes <https://docs.mattermost.com/upgrade/important-upgrade-notes.html>`__ documentation for details.

Upgrade to Mattermost v6.7
--------------------------

Mattermost v6.7 introduces schema changes in the form of a new index. The following notes our test results for the schema changes:

- MySQL 7M Posts - ~17s (instance: db.r5.xlarge)
- MySQL 9M Posts - 2min 12s (instance: db.r5.large)
- Postgres 7M Posts - ~9s  (instance: db.r5.xlarge)

If you want a zero downtime upgrade, you can apply this index prior to doing the upgrade. This is fully backwards-compatible and will not acquire any table lock or affect any existing operations on the table.

- For MySQL: ``CREATE INDEX idx_posts_create_at_id on Posts(CreateAt, Id) LOCK=NONE;``
- For Postgres: ``CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_posts_create_at_id on posts(createat, id);``

Upgrade to Mattermost v6.0
--------------------------

A Mattermost Server v6.0 upgrade will run significant database schema changes that can cause an extended startup time depending on the dataset size. Zero downtime won't be possible for v6.0, but depending on the efforts made during the migration process, it can be minimized to a large extent. 

Running queries prior to the upgrade can also save some downtime. However, some queries can still cause full table locking and require Mattermost to be in read-only mode for the duration of the query.

We strongly recommend that you:

- Set up a maintenance window outside of working hours to mitigate the migration impact. 
- Keep a backup of your database to ensure you can load a previous database snapshot if necessary.
- Upgrade your instance of Mattermost to the latest `Extended Support Release (ESR) <https://docs.mattermost.com/upgrade/extended-support-release.html>`__ first before attempting to run the Mattermost v6.0 upgrade.

.. important::

  Support for Mattermost Server v6.3 `Extended Support Release <https://docs.mattermost.com/upgrade/extended-support-release.html>`_ is coming to the end of its life cycle on October 15, 2022. Upgrading to Mattermost Server v7.1 Extended Support Release or later is recommended.

Upgrading from a previous Extended Support Release to the latest Extended Support Release is supported. Upgrading from v5.31 to v5.37 should take roughly the same amount of time as upgrading from v5.31 to v5.35, then upgrading v5.35 to 5.37. However, an upgrade directly from v5.31 to v5.37 could potentially take hours due to the database schema migrations required for v5.35. Review the :doc:`important-upgrade-notes` for all intermediate versions in between to ensure you’re aware of the possible migrations that could affect your upgrade.

v6.0 database schema migrations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mattermost v6.0 introduces several database schema changes to improve both database and application performance. The upgrade will run significant database schema changes that can cause an extended startup time depending on the dataset size. We've conducted extensive tests on supported database drivers including MySQL and PostgreSQL, using realistic datasets of more than 10 million posts and more than 72 million posts.

A migration to v6.0 of 10+ million posts will take approximately 1 hour and 22 minutes to complete for a MySQL database. See the `Mattermost v6.0 DB schema migrations analysis <https://gist.github.com/streamer45/59b3582118913d4fc5e8ff81ea78b055>`__ documentation for test specifications, data sizes, and test results.

A large migration from v5.39 to v6.0 of 72+ million posts will take approximately 3 hours and 40 minutes to complete for a MySQL database. See the `Migration results analysis <https://gist.github.com/streamer45/868c451164f6e8069d8b398685a31b6e>`__ documentation for test specifications, data sizes, and test results.

.. tabs::

   .. tab:: MySQL

      The following queries, executed during the migration process on an environment with 10+ million posts, will have a significant impact on database CPU usage and write operation restrictions for the duration of the query:

      ``ALTER TABLE Posts MODIFY Props JSON;`` (~26 minutes)

      ``ALTER TABLE Posts DROP COLUMN ParentId;`` (~26 minutes)

      ``ALTER TABLE Posts MODIFY COLUMN FileIds text;`` (~26 minutes)

      For a complete breakdown of MySQL queries, as well as their impact and duration, see the `Mattermost v6.0 DB schema migrations analysis <https://gist.github.com/streamer45/59b3582118913d4fc5e8ff81ea78b055#mysql-1>`__ documentation.

      **MySQL Mitigation Strategies**

      **Run combined queries prior to the upgrade.**
      The previous queries can be combined when run prior to the upgrade as follows:

      ``ALTER TABLE Posts MODIFY COLUMN FileIds text, MODIFY COLUMN Props JSON;``

      This limits the time taken to that of a single query of that type.

      **Online migration**: An online migration that avoids locking can be attempted on MySQL installations, especially for particularly heavy queries or very big datasets (tens of millions of posts or more). This can be done through an external tool like `pt-online-schema-change <https://www.percona.com/doc/percona-toolkit/LATEST/pt-online-schema-change.html>`__. However, the online migration process can cause a significant spike in CPU usage on the database instance it runs.

      See the `Mattermost v6.0 DB schema migrations analysis <https://gist.github.com/streamer45/59b3582118913d4fc5e8ff81ea78b055#online-migration-mysql>`__ documentation for a sample execution and additional caveats.

   .. tab:: PostgreSQL

      The following query executed during the migration process will have a significant impact on database CPU usage and write operation restrictions for the duration of the query:

      ``ALTER TABLE posts ALTER COLUMN props TYPE jsonb USING props::jsonb;`` (~ 11 minutes)

      For a complete breakdown of PostgreSQL queries, as well as their impact and duration, see the `Mattermost v6.0 DB schema migrations analysis <https://gist.github.com/streamer45/59b3582118913d4fc5e8ff81ea78b055#postgresql-1>`__.

Upgrade from releases older than v5.35
----------------------------------------

Customers upgrading from a release older than Mattermost v5.35 should expect extended downtime when upgrading to v6.0 due to the introduction of backend database architecture introduced in v5.35. This upgrade path isn't recommended for large installations. We recommend upgrading to the latest `Extended Support Release (ESR) <https://docs.mattermost.com/upgrade/extended-support-release.html>`__ first before upgrading to Mattermost v6.0. See the `Mattermost changelog <https://docs.mattermost.com/install/self-managed-changelog.html>`__ documentation for additional details.

If you're upgrading from a version prior to Mattermost v5.0, you can't upgrade directly to v6.0. Instead, we strongly recommend approaching the upgrade in phases, starting with an upgrade to the latest ESR first, followed by the upgrade to v6.0. During the first phase of updates, you must also modify your service file to work with the binary changes introduced with the v5.0 release. Your execution directory should point to the Mattermost base directory (i.e. ``/opt/mattermost``), and your binary should point to the ``mattermost`` binary (i.e. ``/opt/mattermost/bin/mattermost``). 

Ensure you review the :doc:`important-upgrade-notes` for all intermediate release versions in between to ensure you’re aware of the possible migrations that could affect your upgrade.

.. note::

  Customers upgrading from releases older than v5.35 following our recommended upgrade process may encounter the following error during the upgrade to v6.0:
  
  ``Failed to alter column type. It is likely you have invalid JSON values in the column. Please fix the values manually and run the migration again.","caller":"sqlstore/store.go:854","error":"pq: unsupported Unicode escape sequence``
  
  To assist with troubleshooting, you can enable ``SqlSettings.Trace`` to narrow down what table and column are causing issues during the upgrade. The following queries change the columns to JSONB format in PostgreSQL. Run these against your v5.39 development database to find out which table and column has Unicode issues:
  
  .. code-block:: sh

    ALTER TABLE posts ALTER COLUMN props TYPE jsonb USING props::jsonb;
    ALTER TABLE channelmembers ALTER COLUMN notifyprops TYPE jsonb USING notifyprops::jsonb;
    ALTER TABLE jobs ALTER COLUMN data TYPE jsonb USING data::jsonb;
    ALTER TABLE linkmetadata ALTER COLUMN data TYPE jsonb USING data::jsonb;
    ALTER TABLE sessions ALTER COLUMN props TYPE jsonb USING props::jsonb;
    ALTER TABLE threads ALTER COLUMN participants TYPE jsonb USING participants::jsonb;
    ALTER TABLE users ALTER COLUMN props TYPE jsonb USING props::jsonb;
    ALTER TABLE users ALTER COLUMN notifyprops TYPE jsonb USING notifyprops::jsonb;
    ALTER TABLE users ALTER COLUMN timezone TYPE jsonb USING timezone::jsonb;

  Once you've identified the table being affected, verify how many invalid occurrences of `\u0000` you have using the following SELECT query:

  .. code-block:: sh

    SELECT COUNT(*) FROM TableName WHERE ColumnName LIKE '%\u0000%';

  Then select and fix the rows accordingly. If you prefer, you can also fix all occurrences at once in a given table or column using the following UPDATE query:

  .. code-block:: sh

    UPDATE TableName SET ColumnName = regexp_replace(ColumnName, '\\u0000', '', 'g') WHERE ColumnName LIKE '%\u0000%';

Upgrade high availability deployments
---------------------------------------

In `high availability <https://docs.mattermost.com/scale/high-availability-cluster.html>`__ environments, you should expect to schedule downtime for the upgrade to v6.0. Based on your database size and setup, the migration to v6.0 can take a significant amount of time, and may even lock the tables for posts which will prevent your users from posting or receiving messages until the migration is complete.

Ensure you review the `high availability cluster upgrade guide <https://docs.mattermost.com/scale/high-availability-cluster.html#upgrade-guide>`__, as well as the :doc:`important-upgrade-notes` to make sure you're aware of any actions you need to take before or after upgrading from your particular version.

.. important::

  We only support one minor version difference between server versions when performing a rolling upgrade. For example v5.27.1 + v5.27.2 or v5.26.4 + v5.27.1 is supported, whereas v5.25.5 + v5.27.0 is not supported. 

  Running two different versions of Mattermost in your cluster should not be done outside of an upgrade scenario. Due to a fundamental change to the clustering code in v6.0, nodes from different versions cannot be run, as noted in the :doc:`important-upgrade-notes` product documentation.

  The release of v6.0 also introduces database schema changes and longer migration times should be expected, especially on MySQL installations. 
