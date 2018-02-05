# Creating Hypertables

The primary point of interaction with your data is a hypertable,
the abstraction of a single continuous table across all space and time intervals, such that one can query it via vanilla SQL.

>ttt First make sure that you have properly [installed][] **AND [setup][]** your Timescale database within your PostgreSQL instance.

### Creating a (Hyper)table
To create a hypertable, you start with a regular SQL table, and then convert
it into a hypertable via the function `create_hypertable` ([API reference][]).

The following example creates a hypertable for tracking
temperature and humidity across a collection of devices over time.

```sql
-- We start by creating a regular SQL table

CREATE TABLE conditions (
  time        TIMESTAMPTZ       NOT NULL,
  location    TEXT              NOT NULL,
  temperature DOUBLE PRECISION  NULL,
  humidity    DOUBLE PRECISION  NULL
);
```

Next, transform it into a hypertable with `create_hypertable`:

```sql
-- This creates a hypertable that is partitioned by time
--   using the values in the `time` column.

SELECT create_hypertable('conditions', 'time');

-- OR you can additionally partition the data on another
--   dimension (what we call 'space partitioning').
-- E.g., to partition `location` into 4 partitions:

SELECT create_hypertable('conditions', 'time', 'location', 4);
```

### Inserting & Querying
Inserting data into the hypertable is done via normal SQL `INSERT` commands,
e.g. using millisecond timestamps:
```sql
INSERT INTO conditions(time, location, temperature, humidity)
  VALUES (NOW(), 'office', 70.0, 50.0);
```

Similarly, querying data is done via normal SQL `SELECT` commands.
```sql
SELECT * FROM conditions ORDER BY time DESC LIMIT 100;
```

SQL `UPDATE` and `DELETE` commands also work as expected. For more
examples of using TimescaleDB's standard SQL interface, please see our
[use pages][].

[installed]: /getting-started/installation
[setup]: /getting-started/setup
[API Reference]: /api#create_hypertable
[use pages]: /using-timescaledb