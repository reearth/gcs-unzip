# gcs-unzip

gcs-unzip is a tool developed in Go for extracting files from archive files on Google Cloud Storage (GCS), downloading the archive file locally, and sequentially uploading the extracted files back to GCS. This approach helps to minimize the required disk space during the extraction process.

## Description

gcs-unzip provides a convenient way to extract files from archive files (such as ZIP or 7Z) stored on GCS. The tool downloads the archive file to the local machine, extracts the files, and then sequentially uploads the extracted files back to GCS. This allows for efficient disk space utilization, as the extracted files are uploaded while the archive is being processed, reducing the amount of local disk space required.

## Features

- Extract files from archive files (ZIP and 7Z) stored on Google Cloud Storage (GCS)
- Downloads the archive file locally and uploads extracted files back to GCS
- Minimizes required disk space during the extraction process
- Efficient and scalable extraction process
- Simple and intuitive command-line interface

## Installation

There are multiple ways to install and use gcs-unzip: using the compiled binary, the container image, or downloading from GitHub Releases.


### Using the Compiled Binary

To install gcs-unzip using the compiled binary, you can use the `go install` command:

```shell
go install github.com/orisano/gcs-unzip@latest
```

Make sure you have Go installed and configured correctly.

### Using the Container Image

Alternatively, you can use the container image provided at `ghcr.io/orisano/gcs-unzip`. This method does not require Go to be installed on your machine.

```shell
docker pull ghcr.io/orisano/gcs-unzip
```

### Downloading from GitHub Releases

You can also download precompiled binaries from the [GitHub Releases](https://github.com/orisano/gcs-unzip/releases) page.

1. Go to the [GitHub Releases](https://github.com/orisano/gcs-unzip/releases) page.
1. Download the binary suitable for your operating system.
1. Place the binary in a directory included in your system's `PATH` environment variable.


## Usage

gcs-unzip is used to extract files from archive files stored on Google Cloud Storage (GCS) and sequentially upload them to GCS.

```shell
gcs-unzip [OPTIONS] <src> <dest>
```

* `<src>`: The source GCS object in the format `<bucket>/<object>`. This specifies the archive file to extract from.
* `<dest>`: The destination GCS prefix in the format `<bucket>/<prefix>`. This specifies the location to upload the extracted files.

```
Options:
  -buf value
    Copy buffer size (default 512k)
  -chunk value
    Upload chunk size (default 16m)
  -disk-limit value
    Disk limit (default 50g)
  -gc int
    Garbage collection interval
  -gcs-meta string
    Metadata (comma separated key=value pairs)
  -gzip-ext string
    Comma-separated list of file extensions to gzip before uploading
    (overrides the built-in compressible list)
  -no-gzip
    Disable gzip compression entirely
  -n int
    Number of goroutines for uploading (default 24)
  -old-windows
    Enable old windows compatibility mode
  -skip-top
    Skip top-level directory in archive
  -tmp-dir string
    Temporary directory
  -v Show verbose output
  -with-meta
    Include metadata files (like .DS_Store, __MACOSX, etc.)
```

## Gzip compression

By default, gcs-unzip uploads files whose extensions appear in a
built-in compressible list (see below) with `Content-Encoding: gzip`.
This behavior can be customized:

- `-gzip-ext js,css,html` — override the built-in list with an explicit
  set of extensions (the leading dot is optional).
- `-no-gzip` — disable gzip compression entirely; every file is
  uploaded as-is.

## Development

### Regenerating the compressible extensions list

`compressible_ext.go` contains the default set of file extensions whose
contents typically benefit from gzip compression. It is generated from
the [`jshttp/mime-db`](https://github.com/jshttp/mime-db) dataset by
filtering entries with `compressible: true`, excluding MIME types whose
payloads are already compressed or are container archives (e.g.
`application/octet-stream`, `application/x-tar`, VM disk images,
Photoshop, DDS textures), and adding a few extra extensions that are
not covered by mime-db but are known to compress well:

- `bin`
- 3D Tiles: `b3dm`, `i3dm`, `pnts`, `cmpt`, `subtree`
- Cesium quantized-mesh terrain: `terrain`

To regenerate the list (e.g. after a `mime-db` update):

```shell
go generate ./...
```

The generator lives in `gen.go` (built with `//go:build ignore`) and
fetches the latest `mime-db` data from the upstream repository.

## License
This project is licensed under the MIT License.
