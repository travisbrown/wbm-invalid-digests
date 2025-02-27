# Invalid digests

The [Wayback Machine][wayback-machine]'s [CDX index][wayback-machine-cdx] provides a `digest` field
that is generally a [Base32-encoded][base32] [SHA-1][sha-1] hash of the content of the snapshot. In
some cases, though, the value of the digest field does not match the expected value. This
repository is intended to document the issue and collect examples of these mismatches.

## Known problems

### Multi-block hashing

For many of these mismatches, the cause seems to be a bug in archiving software used to create the
[Internet Archive][internet-archive]'s [WARC files][warc]. The WARC standard allows the content of
a record to be split into blocks, which are separated by two CRLFs. The problem is that in some
cases, the `WARC-Payload-Digest` is the hash of the bytes of the blocks, including these newlines,
instead of being the hash of the concatenated blocks, which are what is served to the user.

It's possible to verify the CDX hash in this case if you know the original block structure of the
record in the WARC file. In some cases the Internet Archive makes the WARC files available to the
public, but in most cases they don't. For smaller files, it is sometimes possible to reconstruct
the block structure, but my initial experiments suggest that there's too much variation in the size
of the blocks for this to be feasible for large files (at least without a lot of computing power
that I don't have).

As an example, take [this snapshot](https://web.archive.org/web/20170827043307im_/https://altright.com/wp-content/uploads/):

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
<head>
<title>Index of /wp-content/uploads</title>
</head>
<body>
<h1>Index of /wp-content/uploads</h1>
<table>
<tr><th valign="top">&nbsp;</th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
<tr><th colspan="5"><hr></th></tr>
<tr><td valign="top">&nbsp;</td><td><a href="/wp-content/">Parent Directory</a> </td><td>&nbsp;</td><td align="right"> - </td><td>&nbsp;</td></tr>
<tr><td valign="top">&nbsp;</td><td><a href="2015/">2015/</a> </td><td align="right">2017-01-15 20:34 </td><td align="right"> - </td><td>&nbsp;</td></tr>
<tr><td valign="top">&nbsp;</td><td><a href="2016/">2016/</a> </td><td align="right">2017-01-16 00:08 </td><td align="right"> - </td><td>&nbsp;</td></tr>
<tr><td valign="top">&nbsp;</td><td><a href="2017/">2017/</a> </td><td align="right">2017-08-01 06:00 </td><td align="right"> - </td><td>&nbsp;</td></tr>
<tr><th colspan="5"><hr></th></tr>
</table>
</body></html>
```

If you [check the CDX index](https://web.archive.org/cdx/search/cdx?url=https://altright.com/wp-content/uploads/) for this URL, you'll see the following:

```
com,altright)/wp-content/uploads 20170827043307 https://altright.com/wp-content/uploads/ text/html 200 4FEOXGJEZDKTVIMASKG74TAP4C2KQ3FG 997
com,altright)/wp-content/uploads 20171020220258 https://altright.com/wp-content/uploads/ text/html 200 HYUAWLPPGPFV5JB2SLECLYZTCMZCP4TN 765
com,altright)/wp-content/uploads 20171023140440 https://altright.com/wp-content/uploads/ warc/revisit - HYUAWLPPGPFV5JB2SLECLYZTCMZCP4TN 491
com,altright)/wp-content/uploads 20171027162048 https://altright.com/wp-content/uploads/ text/html 200 DQFOFXXD4IVKG5YIJVEPGDGX7O4MDZY6 750
com,altright)/wp-content/uploads 20171027215831 https://altright.com/wp-content/uploads/ text/html 200 6LTY2ERO2FS7CBIWQ7S6H6DMERBAVEZT 795
com,altright)/wp-content/uploads 20171029132803 https://altright.com/wp-content/uploads/ warc/revisit - 6LTY2ERO2FS7CBIWQ7S6H6DMERBAVEZT 490
com,altright)/wp-content/uploads 20171030173903 https://altright.com/wp-content/uploads/ text/html 200 HVERX5YL6WH4XXCIXSO2L3D5H73B6R6I 809
com,altright)/wp-content/uploads 20171031131659 https://altright.com/wp-content/uploads/ warc/revisit - HVERX5YL6WH4XXCIXSO2L3D5H73B6R6I 492
com,altright)/wp-content/uploads 20171103201523 https://altright.com/wp-content/uploads/ text/html 200 KTOCU723VBVKKMHM66DLKJPTD7J3JWNI 817
com,altright)/wp-content/uploads 20171127204054 https://altright.com/wp-content/uploads/ text/html 200 KWDBFIYVYLZ54WJUM5PNFGDFG3YLXKXI 843
com,altright)/wp-content/uploads 20171223193234 https://altright.com/wp-content/uploads/ text/html 200 7UO5MJ7IEPHNQJ2OA5M4RUCNQCA2DVCC 791
com,altright)/wp-content/uploads 20200519145426 https://altright.com/wp-content/uploads/ text/html 403 AACNJ6BLKRQBGQSLFYG6BBBZKBY656ML 761
```

The hash of our downloaded bytes is `KDHJZON5TUIZ5BDN53G4ZGMGKFHERPA3`, which is not what we'd
expect from the first line here, but if we prepend `434\r\n` and append `\r\n0\r\n\r\n` to our
downloaded bytes (where `434` is the original length in hexadecimal), we'll get the expected hash.

### Other issues

I know that this hashing bug explains some of these mismatches, but there are other problems I
don't have an explanation for, including some cases where the CDX `digest` field isn't even a
Base32-encoded SHA-1 hash.

## Contents of this repository

At the moment this repository only includes one data file, which lists examples of snapshots with
invalid CDX hashes. This is a CSV file with four columns: the URL, the WBM timestamp, the expected
hash, and the actual hash. The file is sorted by timestamp and then expected hash.

For now the examples given here are mostly JSON snapshots of tweets, since these tend to be small
files. No claims are made about the contents of these snapshots.

At some point I'll probably try to provide more detailed examples of the bugs I've seen, and also
code for inferring the block structure.

[base32]: https://en.wikipedia.org/wiki/Base32
[internet-archive]: https://archive.org/
[sha-1]: https://en.wikipedia.org/wiki/SHA-1
[warc]: https://iipc.github.io/warc-specifications/specifications/warc-format/warc-1.1/
[wayback-machine]: https://web.archive.org/
[wayback-machine-cdx]: https://github.com/internetarchive/wayback/tree/master/wayback-cdx-server
