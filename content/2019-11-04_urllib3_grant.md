Title: I've been paid to work on Open Source
Date: 2019-11-05T09:00:00+04:00
Category: Logiciel

<figure style="float:right; width: 40%">
  <a href="https://github.com/urllib3/urllib3/issues/1597#issuecomment-495348116">
    <img title="A logo I'd like to see adopted for urllib3 :)" src="{static}/images/tentative-urllib3-logo.png"  style="width: 100%; max-height: 300px; height: auto; padding: 0" /></a>
  <figcaption style="text-align: center; font-size: 0.8rem">A <a href="https://github.com/urllib3/urllib3/issues/1597#issuecomment-495348116">possible</a> urllib3 logo</figcaption>
</figure>

I spent the last five days working full-time on
[urllib3](https://urllib3.readthedocs.io/en/latest/), currently the
[most downloaded Python package](https://pypistats.org/top), and a
dependency of requests. I can’t believe I’m writing this sentence! But
it’s true. A dream come true.  This was made possible thanks to:

 * the urllib3 sponsors,
 * Andrey Petrov who works hard behind the scenes to make this work
 * Seth M. Larson, which reviewed many of my pull requests, including all the
   recent ones

I can’t thank them enough!

Before explaining my work, let me share my urllib3 story. I’ve been
working on urllib3 in my spare time for nearly two years now, in order
to make it work in various async contexts (asyncio, trio, Twisted...).
This work naturally broke many tests, and I spent a lot of time fixing
them! As a result, I’ve become very familiar with urllib3’s test
suite, and submitted a number of improvements upstream when it made
sense. During that time, I got the opportunity to fix the Windows CI,
to fix our test TLS intermediate certificates for macOS 10.13, and
helped to migrate from unittest to pytest.  And also fixed many other
small things.

There were other problems in the test suite, sadly I never got around to fixing
them because of lack of time and energy. But I recently had the opportunity to
spend five days full-time to fix those issues! My experience with the test
suite allowed me to have real impact. Here’s what I did.

## Test timeouts

For each commit, we run about 1k tests in various operating systems (Linux,
Windows, macOS), Python implementations (CPython and PyPy) and Python versions
(Python 2.7 and five Python 3 versions). So that amounts to about 20k tests for
each commit or pull request. That’s a lot of opportunity to get failures!
Especially on CI environments where execution is sometimes really slow. As a
result, we often receive pull requests that fail for those wrong reasons, and
spend a lot of time trying to figure which failures are false negatives. I
submitted a number of pull requests here:

 * [#1717](https://github.com/urllib3/urllib3/pull/1717) unifies
   timeouts values to use larger values in CI and improves various
   tests
 * [#1718](https://github.com/urllib3/urllib3/pull/1718) retries
   failed tests once
 * [#1711](https://github.com/urllib3/urllib3/pull/1711) removes an
   arbitrary value in an existing test
 * [#1729](https://github.com/urllib3/urllib3/pull/1729) fixes a flaky
   test timeout

## Test TLS certificates

Our test TLS certificates are stored in the opaque PEM format and it’s hard to
make them evolve to use more recent best practices. For example, we use
1024-bit RSA keys when some operating systems now require 2048-bit keys. The
consequences are real: I just noticed that the macOS SecureTransport tests no
longer work on macOS 10.15 Catalina because of this, which means we won’t be
able to test from this operating system until this is fixed.

There are a number of details to worry about when generating certs, and it’s
not easy to find the correct OpenSSL incantations to fix our many existing
blobs to adhere to the latest best practices. This is why I worked towards
generating certificates from Python code using the trustme library: this
decouples the parts we actually want to specify like the hostname of the
certificate from the low-level decisions such as the length of the key. We can
delegate the latter decisions to trustme, a library which works well with all
recent operating systems and continues to evolve. The full conversion to
certificates-as-code will take more time, but I did make progress here:

 * [#1721](https://github.com/urllib3/urllib3/pull/1721) removes
   certificates that we no longer use
 * [#1722](https://github.com/urllib3/urllib3/pull/1722) removes a
   directory full of hardcoded certificates
 * [#1725](https://github.com/urllib3/urllib3/pull/1725) generates
   client certs using trustme (which required [importing our root CA
   in trustme](https://github.com/urllib3/urllib3/pull/))

## Test coverage

I brought our code coverage back to 100% in
[#1726](https://github.com/urllib3/urllib3/pull/1726), which uncovered
a bug in the way we parse URLs. I also provided instructions in
[#1727](https://github.com/urllib3/urllib3/pull/1727) to help us
notice pull requests that lower our coverage.

## Code review

I also took advantage of the large chunks of concentration I had to
review two pull requests:

 * [#1715](https://github.com/urllib3/urllib3/pull/1715) where I wrote
   a test which allowed us to merge the bug fix
 * [#1679](https://github.com/urllib3/urllib3/pull/1679) took me about
   a day but I believe my comment will help us understand how this
   pull request fits in the existing proxy support

## Conclusion

And finally, I opened many small pull requests to fix various little
issues: [#1712](https://github.com/urllib3/urllib3/pull/1712),
[#1713](https://github.com/urllib3/urllib3/pull/1713),
[#1716](https://github.com/urllib3/urllib3/pull/1716),
[#1719](https://github.com/urllib3/urllib3/pull/1719),
[#1720](https://github.com/urllib3/urllib3/pull/1720),
[#1723](https://github.com/urllib3/urllib3/pull/1723) and
[#1728](https://github.com/urllib3/urllib3/pull/1728).

All in all, those five days were incredibly productive, and I’m happy
that I was able to put the urllib3 test suite in better shape. All
those small things will make it easier for future contributors to
continue improving urllib3 for the many years to come! I’m proud to be
part of this community.

<!-- vim: spelllang=en
-->
