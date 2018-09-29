# rupdater

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/c1d9a929a0004ed28d7f447067dd2287)](https://app.codacy.com/app/justr4nx/rupdater?utm_source=github.com&utm_medium=referral&utm_content=r4nx/rupdater&utm_campaign=Badge_Grade_Settings)

rupdater is a super lightweight and dependency-free updater, that provide
remote version parsing, downloading and getting hash sum of an update.

## Installation

rupdater can be install from PyPi by running:

```bash
pip install rupdater
```

## Usage

First of all, create new instance of Updater. It takes 2 arguments:
current local version and URL of the remote version data file:
```python
from rupdater import Updater
updater = Updater('1.0.0a0', 'https://example.com/remote_version.json', use_json=True)
```

Now you can check for an update by calling the `check()` method:
```python
if updater.check():
    print('Update found!')
```

After checking, you may want to download the update. This can be done
with `download()` method, that returns a file-like object (byte-mode)
of the downloaded update (it stores in a temp file).
You should use it only with context manager.

```python
with updater.download() as update_file, open('update.zip', 'w+b') as f:
    f.write(update_file.read())
```

This example will read the data from update_file and write it to the update.zip.

More complicated example with chunk-by-chunk reading/writing:
```python
with updater.download() as update_file, open('update.zip', 'w+b') as f:
    for chunk in iter(lambda: update_file.read(1024 * 16), b''):
        f.write(chunk)
```

rupdater also has hash static method, that takes 2 required arguments and 1 optional:
file-like object, hashing algorithm (as string) and chunk size, which is
1024 * 16 by default. This is the example of usage:
```python
with updater.download() as update_file:
    hashes_match = Updater.hash_file(update_file, updater.hash_algo) == updater.hash
```

Warning! If you want to reuse the update file, you have to `seek(0)`. Example:
```python
with updater.download() as update_file, open('update.zip', 'w+b') as f:
    for chunk in iter(lambda: update_file.read(1024 * 16), b''):
        f.write(chunk)
    f.seek(0)
    assert Updater.hash_file(f, updater.hash_algo) == updater.hash
```

You can also parse version data again manually by calling `get_version_data()`.

## Portability
`rupdater.py` is absolutely independent of anything at the moment.
Python Standard Library is the only requirement. So if you need,
you can just copy this file and use it like a single module.

## Exceptions handling
rupdater does not catch any errors, you have to do it yourself as prefer.
In addition, rupdater may raise ValueError when incorrect input data was given
(version data file does not formatted properly, unknown hashing algorithm etc).
