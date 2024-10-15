Title: Fixing SSLV3_ALERT_HANDSHAKE_FAILURE with urllib3 2.0
Date: 2023-11-02T11:00:00+04:00
Category: Logiciel

<figure style="float:right; width: 40%">
    <img title="urllib3 logo" src="{static}/images/urllib3-logo.svg" style="width: 100%; max-height: 300px; height: auto; padding: 0" />
</figure>


Over the years, the urllib3 HTTP client maintained its SSL cipher suites based on Hynek's [evergreen list](https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/). However, with the release of urllib3 version 2.0, we decided to require OpenSSL 1.1.1 or above and rely on the default cipher suites, which are secure starting with OpenSSL 1.1.1. The shift aimed to eliminate the need for maintaining a separate list and align with secure defaults. (However, we realized after the release that CPython was also defining defaults based on Hynek's list! This means that we currently rely on Python rather than OpenSSL, but the result is the same.) Importantly, this adjustment aligns with the future as TLS 1.3 supports only 5 cipher suites, which is likely to eliminate the problem of choosing cipher suites in the future.

Despite this improvement, users have reported `SSLV3_ALERT_HANDSHAKE_FAILURE` errors since the April release. This error, cryptic but common in OpenSSL, signifies a lack of common cipher suites between the server and the client.

## Step by step example

What usually happens is that the server only supports a few weak cipher suites and you need to ask urllib3 to support one of those. Let's take as an example [urllib3#3059](https://github.com/urllib3/urllib3/issues/3059) where access to api.etrade.com was failing. Here's how to fix the issue step by step:

1. Identify the supported cipher suites. Use tools like SSL Labs for public sites or testssl.sh for private sites. At the time of writing in November 2023, [api.etrade.com only supported the following weak cipher suites](https://www.ssllabs.com/ssltest/analyze.html?d=api.etrade.com&latest):
    * `TLS_RSA_WITH_AES_256_GCM_SHA384` (0x9d)
    * `TLS_RSA_WITH_AES_128_GCM_SHA256` (0x9c)
    * `TLS_RSA_WITH_AES_256_CBC_SHA256` (0x3d)
    * `TLS_RSA_WITH_AES_128_CBC_SHA256` (0x3c)
    * `TLS_RSA_WITH_AES_128_CBC_SHA` (0x2f)
    * `TLS_RSA_WITH_AES_256_CBC_SHA` (0x35)

2. Choose a cipher suite, considering security. For example, `TLS_RSA_WITH_AES_256_GCM_SHA384` is a good choice due to its GCM mode and 256-bit encryption.

3. Determine the OpenSSL name for the chosen suite. Indeed, the above are IANA names. Use an [online mapping](https://testssl.sh/openssl-iana.mapping.html) or run `openssl ciphers -s -V`. For our example, `TLS_RSA_WITH_AES_256_GCM_SHA384` corresponds to `AES256-GCM-SHA384` (hex code `0x9d` is formatted as `0x00,0x9D` in the `openssl ciphers` output).

4. Configure urllib3 accordingly. Create a [custom SSL context](https://urllib3.readthedocs.io/en/stable/advanced-usage.html#custom-ssl-contexts) and set the chosen cipher suite per the following examples.

## Code examples

If using urllib3:

```python3
import ssl

from urllib3 import PoolManager
from urllib3.util import create_urllib3_context

ctx = create_urllib3_context()
ctx.load_default_certs()
ctx.set_ciphers("AES256-GCM-SHA384")

with PoolManager(ssl_context=ctx) as pool:
    pool.request("GET", "https://api.etrade.com/…/")
```

If using requests, you need an HTTPAdapter (thanks [@Grub4K](https://github.com/Grub4K) for the tip in [urllib3#3173](https://github.com/urllib3/urllib3/issues/3173)):

```python3
class CustomSSLContextHTTPAdapter(requests.adapters.HTTPAdapter):
    def __init__(self, ssl_context=None, **kwargs):
        self.ssl_context = ssl_context
        super().__init__(**kwargs)

    def init_poolmanager(self, connections, maxsize, block=False):
        self.poolmanager = urllib3.poolmanager.PoolManager(
            num_pools=connections, maxsize=maxsize,
            block=block, ssl_context=self.ssl_context)

session = requests.session()
session.adapters.pop("https://", None)
session.mount("https://", CustomSSLContextHTTPAdapter(ctx))
```

If you face any issues, [ask in our community Discord](https://discord.gg/urllib3).
