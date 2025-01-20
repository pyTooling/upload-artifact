[![GitHub Workflow - Build and Test Status](https://img.shields.io/github/actions/workflow/status/pyTooling/upload-artifact/.github%2Fworkflows%2FArtifactsUpload.yml?branch=dev&logo=githubactions)](https://GitHub.com/pyTooling/upload-artifact/actions/workflows/ArtifactsUpload.yml)
[![Sourcecode License](https://img.shields.io/badge/code-MIT%20License-green?longCache=true&style=flat-square&logoColor=fff)](LICENSE.md)

# Artifact Upload Action with File Permission Preservation

This composite action, based on [`actions/upload-artifact`](https://github.com/actions/upload-artifact) and packaging
the artifact's content in a tarball, will preserve file attributes like **file permissions**. This essential capability
is not implemented by GitHub until now (requested on [05.12.2019](https://github.com/actions/upload-artifact/issues/38))
and still delayed and/or rejected(?) to be implemented in the future. According to GitHub, the internal API doesn't
allow  the implementation of such a feature, but this actions is demonstrating a working solution.

📥 See [pyTooling/download-artifact](https://github.com/pyTooling/download-artifact) for the matching download action.


## Additional Features compared to `actions/upload-artifact`

* Select an operation mode of `tar` or `legacy`. The latter will collect the files in an artifact without a tarball.
* Enabled debug mode, to list all gathered files in an artifact.
* Override the tarballs name in case of naming collisions.


## Advantages Compared to Competing GitHub Actions

* Support all parameters of `actions/upload-artifact`.  
	(Others support only a subset.)
* Supports Ubuntu, Windows and macOS GitHub Runner images.  
	(Others are created for Linux in mind.)
* Well-defined behavior of tarball internal directory/file structure.  
	(No silent and "unpredictable" removal of common prefixes.)
* Store files in tarball without owner and group information.
* Testcases are implemented as a pipeline, verifying uploads/downloads using a tarball.


## Usage

```yaml
jobs:
  MyJob:
    steps:
      - name: 📤 Upload artifact 'binaries'
        uses: pyTooling/upload-artifact@v4
        with:
          name: binaries
          working-directory: build
          path: |
            bin
            lib
            
      - name: 📤 Upload artifact 'logfiles'
        uses: pyTooling/upload-artifact@v4
        with:
          name: logfiles
          path: |
            *.report
            build/*.log
```


### Input Parameters

| Parameter              | Required | Default             | Description                                                                                                                                                                                                                                                                                 |
|------------------------|:--------:|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`                 |    no    | `'artifact'`        | Name of the artifact to upload.                                                                                                                                                                                                                                                             |
| `working-directory`    |    no    | `''`                |                                                                                                                                                                                                                                                                                             |
| `path`                 |   yes    |                     | A list of files, directories or wildcard patterns that describes what to upload.                                                                                                                                                                                                            |
| `if-no-files-found`    |    no    | `'warn'`            | The desired behavior if no files are found using the provided path.<br/>Available Options:<br/> • `warn`: Output a warning but do not fail the action<br/> • `error`: Fail the action with an error message<br/> • `ignore`: Do not output any warnings or errors, the action does not fail |
| `retention-days`       |    no    | repository settings | Duration after which artifact will expire in days. 0 means using default retention. <br/> Minimum 1 day.<br/> Maximum 90 days unless changed from the repository settings page.                                                                                                             |
| `compression-level`    |    no    | `6`                 | The level of compression for Zlib to be applied to the artifact archive.<br/> The value can range from 0 to 9.<br/> For large files that are not easily compressed, a value of 0 is recommended for significantly faster uploads.                                                           |
| `overwrite`            |    no    | `false`             | If true, an artifact with a matching name will be deleted before a new one is uploaded.<br/> If false, the action will fail if an artifact for the given name already exists.<br/> Does not fail if the artifact does not exist.                                                            |
| `include-hidden-files` |    no    | `false`             | Whether to include hidden files in the provided path in the artifact.<br/> The file contents of any hidden files in the path should be validated before enabled this to avoid uploading sensitive information.                                                                              |
| `mode`                 |    no    | `'tar'`             | Mode of operation. Allowed modes:<br/> • `tar` (default),<br/> • `legacy`                                                                                                                                                                                                                   |
| `investigate`          |    no    | `false`             | If enabled, list content of the created tarball.                                                                                                                                                                                                                                            |
| `tarball-name`         |    no    | [^1]                | Filename of the embedded tarball.                                                                                                                                                                                                                                                           |

[^1]: `'__pyTooling_upload_artifact__.tar'`

### Output Parameters

| Parameter      | Description                                                                                                                                                                                                                                                                                    |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `artifact-id`  | GitHub ID of an Artifact, can be used by the REST API.                                                                                                                                                                                                                                         |
| `artifact-url` | URL to download an Artifact. Can be used in many scenarios such as linking to artifacts in issues or pull requests. Users must be logged-in in order for this URL to work. This URL is valid as long as the artifact has not expired or the artifact, run or repository have not been deleted. |


## Fixed behavior compared to `actions/upload-artifact`

1. **Preserve file permissions**  
   The artifact's content is collected in a tarball, which allows preserving file attributes like file permissions.
2. **Don't remove common prefix from files**  
   `actions/upload-artifact` removes the common prefix from all files before storing in an artifact. This is not a
   well-defined behavior. Slightly changing the list of collected files might drastically change the directory structure
   of the artifact.  
   This action defines a root directory from where the content of the tarball is constructed. This is independent of the
   list of provided file patterns.

## Further Features

* Files can be uploaded as a legacy artifact using `actions/upload-artifact`.  
  `pyTooling/download-artifact` accepts artifacts from `actions/upload-artifact`, `pyTooling/upload-artifact` *tar* mode
  and `pyTooling/upload-artifact` *legacy* mode.


## Limitations of `tar`

This composite action uses `tar`/`gtar` (GNU tar) as provided by the GitHub runner's operating system images.

### On Linux, macOS and Windows (GNU tar)

To ensure files starting with a dash aren't considered command line options to `tar`, `tar` is called with
`--verbatim-files-from` option.

To ensure files are extracted and assigned to the owner/group of the extracting user, options `--owner=root:0` and
`--group=root:0` are used when creating the tarball.

In case, parameter `include-hidden-files` isn't set, hidden files (dot-files) are removed from the tarball in a further
cleanup step using the `--delete` command. This step is needed to ensure compatibility with the original
`actions/upload-artifact` action provided by GitHub.

### Alternative BSD tar on macOS

BSD tar has even worse limitations than GNU tar and offers fewer features. Thus, this composite action uses GNU tar on
macOS, too. Fortunately, GNU tar (`gtar`) is already preinstalled via homebrew on all macOS runner images provided by
GitHub. For more details on BSD tar expand the following collapsible section.

<details><summary>Unused BSD tar on macOS </summary>
⚠ BSD tar doesn't support a `--delete` option. Thus, hidden files (dot files) can't be removed (excluded) from tarballs.
Removing discovered hidden files afterward from created tarballs is used on runner OS providing GNU tar. This technique
can't be applied to BSD tar. [^2]

⚠ BSD tar doesn't support a `--verbatim-files-from` option. Thus, files starting with a dash might be interpreted by `tar`
as a command line option.

> **-T filename**, **--files-from filename**  
> In `x` or `t` mode, tar will read the list of names to be extracted from filename. In `c` mode, tar will read names to
> be archived from filename. The special name `-C` on a line by itself will cause the current directory to be changed to
> the directory specified on the following line. Names are terminated by newlines unless `--null` is specified. Note
> that `--null` also disables the special handling of lines containing `-C`.  
> Note: If you are generating lists of files using `find(1)`, you probably want to use `-n` as well.
> 
> Source: https://man.freebsd.org/cgi/man.cgi?tar(1)

To ensure files are extracted and assigned to the owner/group of the extracting user, options `--gname=root`, `--gid=0`,
`--uname=root` and `--uid=0` are used when creating the tarball.
</details>

## Dependencies

* [actions/upload-artifact@v4](https://github.com/actions/upload-artifact)


## Competing Actions

* [eviden-actions/upload-artifact](https://github.com/eviden-actions/upload-artifact)
* [nmerget/upload-gzip-artifact](https://github.com/nmerget/upload-gzip-artifact)
* [alehechka/upload-tartifact](https://github.com/alehechka/upload-tartifact)


## Contributors

* [Patrick Lehmann](https://GitHub.com/Paebbels) (Maintainer)
* [Sven Köhler](https://GitHub.com/skoehler)
* [and more...](https://GitHub.com/pyTooling/upload-artifact/graphs/contributors)

### Credits

**This action was inspired by and is based on:**
 * [actions/upload-artifact#38 - upload-artifact does not retain artifact permissions (08. Sep. 2024)](https://github.com/actions/upload-artifact/issues/38#issuecomment-2336484584)
 * [Gist: 
rcdailey/download-tar-action.yml](https://gist.github.com/rcdailey/cd3437bb2c63647126aa5740824b2a4f)


## License

This GitHub Composite Action (source code) licensed under [The MIT License](LICENSE.md).

---

SPDX-License-Identifier: MIT
