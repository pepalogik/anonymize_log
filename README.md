# anonymize_log

## Purpose

The purpose of the `anonymize_log.py` script is to make access logs of the
[Apache web server](https://httpd.apache.org/) compliant with
[GDPR](https://eur-lex.europa.eu/eli/reg/2016/679/oj) while keeping them
processable by log analysis software.

A typical user will be a web server administrator in the EU, who wants to
generate visitor statistics from access logs and does not want to break the
law.

## Disclaimer

I am not a lawyer, and I have not even read the whole GDPR. I read some
excerpts, explanations, and discussions on the web, and I wrote the script
according to my best understanding of the topic. I do not provide any warranty
on the script or on the information in this document (see also Section 15 of
the license).

If you want to make sure that your log-processing procedures are
GDPR-compliant, please consult a lawyer.

## Status

I wrote the script because I needed it, but I am not planning to further
develop it. I have published it on GitHub, because I think it may be useful for
more people. I am ready to watch issues and pull requests, and I will likely
implement simple bug fixes and accept reasonable pull requests. However, it
will probably be best if someone forks the project and continues its
development independently.

Presumably, the main limitation of the present script is its support of only
one log format, [Apache
Combined](https://httpd.apache.org/docs/1.3/logs.html#combined).

## How it works

GDPR puts restrictions on the handling of personal data. `anonymize_log.py`
removes personal data from access logs and thus also lifts those restrictions
from handling of the logs.

The script removes two types of personal data:

### 1. IP addresses (or hostnames)

IP addresses or hostnames of the clients are replaced by pseudonyms in the form
_hash_._TLD_, where

* _hash_ is the MD5 hash of the hostname (or of the IP address if no hostname
  is available) with a cryptographic salt appended,
* _TLD_ is the actual top-level domain (or "ip" if unknown).

The use of a hash function effectively hides real IP addresses but keeps it
possible to

* identify requests from the same host,
* identify requests from a specified host (by a known IP address). This may be
  useful for excluding specific hosts from statistical analysis.

Maintaining real TLDs is vital for statistical analysis—you can estimate how
many visitors come from each country.

### 2. Query strings in referrers

By "referrer", I mean the URL of the referring page (if any), as stored in the
access log. The query string (the part starting with `?`) may occasionally
contain personal data (such as if a user of a badly designed webmail interface
opens a link in an email). `anonymize_log.py` therefore removes query strings
from referrers, except for search queries at known search engines. (It is
useful to know the search queries that led to your website, and they hopefully
shouldn't contain the visitors' personal data. Unfortunately, Google prevents
queries from being sent within referrers—you have to use the [Google Search
Console](https://search.google.com/search-console/about).)

## How to use it

1. Download `anonymize_log.py`. You do not need the readme and license files,
   because the script itself includes a brief readme and a reference to the
   license.
2. Install Python 3 on your server if it is not installed yet.
3. Read the readme included in the script (it's in the beginning). It contains
   more detailed technical information.
4. Write a control script (see below).
5. Add the control script to your `crontab`.

## Control script

`anonymize_log.py` will not do all the necessary log management—it will only
read a log from stdin and send its anonymized version to stdout. You have to
write a control script that will be specific for your server. Here I give a few
hints.

First of all, the raw, non-anonymized access logs should be stored very
securely and only for a short time. As I understand GDPR, you may process
personal data without the subjects' consent only to the extent necessary for
your legitimate interests, such as resolving technical problems or attacks on
your server. Only those persons who would really be solving the technical
problems may have access to the raw logs.

Then, for the IP address pseudonymization to be really irreversible, you should
always use a cryptographic salt. The salt should be randomly generated,
different for each website (if there are several on the server), and changed
over time. You can generate a random salt with this command (in `bash`):

```bash
SALT="$(dd if=/dev/urandom bs=8 count=1 status=none | base64)"
```

Finally, used salts should not be stored any longer than necessary.

The rest is up to you, good luck!

## Author

Jan Lachnitt
