# syslog Parser Plugin

The `syslog` parser plugin parses syslog generated logs. This plugin
supports two RFC formats, rfc3164 and rfc5424.


Parameters
----------

### time\_format

Specify time format for event time. Default is "%b %d %H:%M:%S" for
rfc3164 protocol.

### message\_format

Specify protocol format. Supported values are `rfc3164`, `rfc5424` and
`auto`. Default is `rfc3164`. If your syslog uses `rfc5424`, use
`rfc5424` instead.

`auto` is useful when this parser receives both `rfc3164` and `rfc5424`
message. `syslog` parser detects message format by using message prefix.

This parameter is used inside `in_syslog` plugin because the file logs
via syslog don't have `<9>` like priority prefix.

### with\_priority

If the incoming logs have priority prefix, e.g. \<9\>, set `true`.
Default is `false`.

### keep\_time\_key

If you want to keep time field in the record, set `true`. Default is
`false`.

Regexp patterns
---------------

### rfc3164 pattern

``` {.CodeRay}
format /^\<(?<pri>[0-9]+)\>(?<time>[^ ]* {1,2}[^ ]* [^ ]*) (?<host>[^ ]*) (?<ident>[a-zA-Z0-9_\/\.\-]*)(?:\[(?<pid>[0-9]+)\])?(?:[^\:]*\:)? *(?<message>.*)$/
time_format "%b %d %H:%M:%S"
```

`pri`, `host`, `ident`, `pid` and `message` are included in the event
record. `time` is used for the event time.

`pri` value is converted into integer type.

If `with_priority` is `false`, `^\<(?<pri>[0-9]+)\>` is removed from the
pattern.

### rfc5424 pattern

``` {.CodeRay}
format /\A^\<(?<pri>[0-9]{1,3})\>[1-9]\d{0,2} (?<time>[^ ]+) (?<host>[^ ]+) (?<ident>[^ ]+) (?<pid>[-0-9]+) (?<msgid>[^ ]+) (?<extradata>(\[(.*)\]|[^ ])) (?<message>.+)$\z/
time_format "%Y-%m-%dT%H:%M:%S.%L%z"
```

`pri`, `host`, `ident`, `pid`, `msgid`, `extradata` and `message` are
included in the event record. `time` is used for the event time.

`pri` value is converted into integer type.

Example
-------

### rfc3164 log

``` {.CodeRay}
<6>Feb 28 12:00:00 192.168.0.1 fluentd[11111]: [error] Syslog test
```

This incoming event is parsed as:

``` {.CodeRay}
time:
1362020400 (Feb 28 12:00:00)

record:
{
  "pri"    : 6,
  "host"   : "192.168.0.1",
  "ident"  : "fluentd",
  "pid"    : "11111",
  "message": "[error] Syslog test"
}
```

### rfc5424 log

``` {.CodeRay}
<16>1 2013-02-28T12:00:00.003Z 192.168.0.1 fluentd 11111 ID24224 [exampleSDID@20224 iut="3" eventSource="Application" eventID="11211"] Hi, from Fluentd!
```

This incoming event is parsed as:

``` {.CodeRay}
time:
1362052800 (2013-02-28T12:00:00.003Z)

record:
{
  "pri"      : 16,
  "host"     : "192.168.0.1",
  "ident"    : "fluentd",
  "pid"      : "11111",
  "msgid"    : "ID24224",
  "extradata": "[exampleSDID@20224 iut=\"3\" eventSource=\"Application\" eventID=\"11211\"]",
  "message"  : "Hi, from Fluentd!"
}
```


------------------------------------------------------------------------

If this article is incorrect or outdated, or omits critical information,
please [let us know](https://github.com/fluent/fluentd-docs/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud
Native Computing Foundation (CNCF)](https://cncf.io/). All components
are available under the Apache 2 License.