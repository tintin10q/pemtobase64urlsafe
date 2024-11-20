# Problem

How can you extract the 32 private key bytes from a PEM file?

# Solution

1. Save your key in pem format to `private.pem` 
2. Run the following command: `openssl pkey -inform pem -in private.pem -text`
3. Now copy the part after `priv:`
4. Use the following script to get the key bytes as a base64urlsafe encoded string

```py
yourkey = """
    00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:
    00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:
    00:00
"""
import base64
key = yourkey.replace("\n","").replace(" ","").split(":")
key_ints = [int(k,16) for k in key]
key_bytes = bytes(key_ints)
key_base64_urlsafe =  base64.urlsafe_b64encode(key_bytes)
key_base64_urlsafe = key_base64_urlsafe.replace(b"=", b"")
print(key_base64_urlsafe)
```


# Background

When working with [Vapid](https://blog.mozilla.org/services/2016/08/23/sending-vapid-identified-webpush-notifications-via-mozillas-push-service/) for the [web push protocol](https://web.dev/articles/push-notifications-web-push-protocol) you have to generate a public and private key pair on the P-256 NIST curve. 
The actual private key is a 256 bit or 32 byte long. If you encode this into base 64 you get a string of 43 characters.

Now many things I use actually want to encode this P-256 into the pem format. That is the format where you have `-----BEGIN PRIVATE KEY-----` and then some lines of text and then `-----END PRIVATE KEY-----` with a final new line.
This format encodes more then just the private key bytes and also for instance what type of key it is or which curve it is for.

Sometimes libraries can deal with this pem format but sometimes you need to give the actual 32 private key bytes. Usually the key bytes need to be given in base64 encoding. 

I only had the pem version. How do you get the 32 key bytes out?

[This great stack overflow awnser](https://stackoverflow.com/questions/77244714/how-can-i-extract-the-32-byte-ed25519-public-key-from-a-pem-file-and-how-can-i/77248795#77248795) told me how to do it!

I am going to copy some of it verbatim:

OpenSSL provides a built-in method to print out the 32-byte hex keys for both private and public keys, whether they are in the DER or PEM format:

    openssl pkey [-pubin] [-inform pem/der] -in [KEYFILE] [-noout] -text

The main options are:

    -pubin: By default a private key is read from the input file: with this option a public key is read instead.
    -inform DER|PEM: This specifies the input format DER or PEM. The default format is PEM.
    -in filename: This specifies the input filename to read a key from.
    -noout: Do not output the encoded version of the key.
    -text: Prints out the various public or private key components in plain text.

See the OpenSSL pkey [documentation](https://docs.openssl.org/1.1.1/man1/pkey/) for more details.
