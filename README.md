# npmjs-offline-install

Windows 11 batch script for downloading npm packages and their runtime
dependencies into a local offline repository when direct `npm install` access to
the public registry is blocked by an internal proxy.

The downloader uses PowerShell HTTP requests with browser-like headers for
registry metadata and package tarball downloads. It does not call `npm` while
collecting packages.

## Usage

```bat
download-npm-offline.bat package[@version-or-range] [output-dir] [registry-url]
```

Examples:

```bat
download-npm-offline.bat express@4 offline-npm-repo
download-npm-offline.bat @types/node@latest offline-npm-repo https://registry.npmjs.org
```

## Output

The output directory is a simple repo that can be copied to an offline machine:

```text
offline-npm-repo/
  metadata/              registry metadata JSON used for resolution
  tarballs/              downloaded .tgz packages named from package.json dependencies
    express-4.18.3.tgz
    @types/
      node-20.19.9.tgz
  package-list.json      downloaded package manifest
  install-offline.bat    helper that seeds npm cache and installs the root package
```

On the offline machine, run:

```bat
offline-npm-repo\install-offline.bat
```

## Notes

- Runtime `dependencies` and `optionalDependencies` are downloaded recursively.
- `devDependencies` are not downloaded.
- Common npm ranges such as exact versions, dist-tags, `^`, `~`, comparison
  ranges, and wildcards are supported.
- If your environment requires a registry mirror, pass it as the third argument.

## Included Electron package

This repository includes an `npm install electron` result for `electron@43.2.0`
with the Windows x64 Electron runtime already downloaded. The completed archive
is split into Git-friendly parts under `archives/`:

```text
archives/electron-43.2.0-npm-install.tar.gz.part-aa
archives/electron-43.2.0-npm-install.tar.gz.part-ab
archives/electron-43.2.0-npm-install.tar.gz.sha256
```

On Windows, restore the archive and extract it with:

```bat
cd archives
copy /b electron-43.2.0-npm-install.tar.gz.part-aa+electron-43.2.0-npm-install.tar.gz.part-ab electron-43.2.0-npm-install.tar.gz
certutil -hashfile electron-43.2.0-npm-install.tar.gz SHA256
tar -xzf electron-43.2.0-npm-install.tar.gz -C ..
```

Compare the `certutil` hash with
`archives/electron-43.2.0-npm-install.tar.gz.sha256`, then use the restored
`package.json`, `package-lock.json`, and `node_modules` contents offline.

## Included ajv and undici packages

`archives/ajv-8.20.0-undici-8.11.2-offline.tar.gz` bundles
[`ajv@8.20.0`](https://github.com/ajv-validator/ajv) and
[`undici@8.11.2`](https://github.com/nodejs/undici) with all of their runtime
dependencies:

```text
ajv-undici-offline/
  package.json           ajv and undici pinned to exact versions
  package-lock.json
  node_modules/          ready-to-use npm install result
  tarballs/              original registry .tgz files
    ajv-8.20.0.tgz
    fast-deep-equal-3.1.3.tgz
    fast-uri-3.1.8.tgz
    json-schema-traverse-1.0.0.tgz
    require-from-string-2.0.2.tgz
    undici-8.11.2.tgz
  install-offline.bat    seeds the npm cache from tarballs/ and runs npm install --offline
```

On Windows, verify and extract it with:

```bat
cd archives
certutil -hashfile ajv-8.20.0-undici-8.11.2-offline.tar.gz SHA256
tar -xzf ajv-8.20.0-undici-8.11.2-offline.tar.gz -C ..
```

Compare the hash with `archives/ajv-8.20.0-undici-8.11.2-offline.tar.gz.sha256`.
Then either copy `ajv-undici-offline\node_modules` into your project, or run
`ajv-undici-offline\install-offline.bat` to reinstall from the bundled tarballs.
To add them to an existing project, run
`npm install --offline <path>\ajv-undici-offline\tarballs\ajv-8.20.0.tgz <path>\ajv-undici-offline\tarballs\undici-8.11.2.tgz`
after seeding the cache with `npm cache add` for each `.tgz`.

`undici@8` requires Node.js 22.19.0 or newer.
