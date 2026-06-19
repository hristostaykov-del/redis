INFO

INFO [section [section ...]]
Description:
The command returns information and statistics about the server, grouped into named sections. The `stream` section reports server-wide, per-database distribution summaries for streams and consumer groups. Each metric is a base-2 logarithmic histogram, identical in form to the `keysizes` section, and is maintained incrementally as commands execute, so reading it is a cheap lookup of already-computed counters with no scans or per-stream enumeration. Collection is controlled by the stream-stats configuration directive; when it is disabled the instrumentation carries no overhead and the section is empty.
Options:
section [section ...] - One or more section names to return - Default: the `default` set of sections. The `stream` value selects the stream statistics section; it is included by `everything` but is not part of `default`, so it is requested explicitly or via `INFO everything`.
stream - Returns the stream statistics section, consisting of seven per-database histograms. The section is populated only when the stream-stats configuration directive is enabled otherwise it contains no lines. Each histogram is rendered as a comma-separated list of `bin=count` pairs, where `bin` is the cumulative upper bound of a base-2 bucket (`1`, `2`, `4`, `8`, ... `1K`, `2K`, ... `64K`) and `count` is the number of streams or consumer groups in that bucket. A metric line is present only when its histogram is non-empty. 
The seven metrics are:
1. distrib_streams_entries - Gauge. Each sample is one stream's current entry count (stream size). Reconstructed from RDB on load and updated in realtime.
2. distrib_streams_memory_bytes - Gauge. Each sample is one stream key's current total allocated memory in bytes. Reconstructed from RDB on load. Present only when memory usage tracking is enabled; the line is absent otherwise.
3. distrib_cgroups_pel - Gauge. Each sample is one consumer group's pending-entry-list size (delivered but unacknowledged messages). Reconstructed from RDB on load. Available only for XREADGROUP commands.
4. distrib_cgroups_lag - Gauge. Each sample is one consumer group's backlog of unconsumed messages (producer position minus group read position). Reconstructed from RDB on load. Available only for XREADGROUP commands.
5. distrib_delivery_latency_ms - Counter. Each sample is one message's first-delivery latency in milliseconds, calculated by subtracting the millisecond timestamp embedded in the message ID from the time of the first read. Accumulates from server start; not reconstructed from RDB. Note: if clients use custom IDs that do not follow the standard <milliseconds>-<sequence> format, the extracted timestamp will not reflect the actual production time and this metric will be inaccurate.
6. distrib_streams_throughput_rps - Rate. Each sample is one stream's messages read per second over a one-second window. Not reconstructed from RDB; present once a full window has elapsed.
7. distrib_streams_bandwidth_bps - Rate. Each sample is one stream's bytes delivered per second over a one-second window. Not reconstructed from RDB; present once a full window has elapsed.

stream-stats <yes | no> - Configuration directive that enables collection of the stream statistics reported by the stream section - Default: no. Settable with CONFIG SET. When disabled, no stream instrumentation runs and the section is empty. When enabled, the gauge metrics (entries, memory, PEL, lag) are accurate immediately if the directive was set at startup or after an RDB load; the rate and counter metrics (throughput, bandwidth, delivery latency) begin accumulating from the moment collection is active and appear once the relevant traffic has flowed.
Response:
Returns a bulk string (RESP2) or verbatim string (RESP3) containing the `# Stream` header followed by one `field:value` line per non-empty metric per database. Field names are prefixed with `db<N>_`, and each value is a comma-separated list of `bin=count` pairs. Databases with no streams contribute no lines.

INFO stream response (single database with stream activity):

127.0.0.1:6379> INFO stream
# Stream
db0_distrib_streams_entries:1=20,8=5,1K=2,64K=1
db0_distrib_streams_memory_bytes:128=12,1K=6,16K=2,256K=1
db0_distrib_cgroups_pel:1=5,2=3,8=1
db0_distrib_cgroups_lag:4=2,16=1,64=3
db0_distrib_delivery_latency_ms:0=100,1=50,2=20,4=5
db0_distrib_streams_throughput_rps:1=10,2=5,4=3,8=1
db0_distrib_streams_bandwidth_bps:1K=5,2K=3,4K=2,64K=1

INFO stream response (multiple databases):

127.0.0.1:6379> INFO stream
# Stream
db0_distrib_streams_entries:1=20,8=5,1K=2
db0_distrib_cgroups_lag:4=2,16=1
db5_distrib_streams_entries:64K=1,256K=1
db5_distrib_cgroups_lag:1K=1

INFO stream response (immediately after restart -- gauges reconstructed from RDB, rate and counter metrics still warming up so their lines are absent):

127.0.0.1:6379> INFO stream
# Stream
db0_distrib_streams_entries:1=20,8=5,1K=2,64K=1
db0_distrib_streams_memory_bytes:128=12,1K=6,16K=2,256K=1
db0_distrib_cgroups_pel:1=5,2=3
db0_distrib_cgroups_lag:4=2,16=1,64=3
