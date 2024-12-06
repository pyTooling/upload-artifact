# Preserving Artifact Upload Action

Upload artifacts and preserve file attributes like file permissions.


**Based on:**
 * [actions/upload-artifact#38 - upload-artifact does not retain artifact permissions (08. Sep. 2024)](https://github.com/actions/upload-artifact/issues/38#issuecomment-2336484584)
 * [Gist: 
rcdailey/download-tar-action.yml](https://gist.github.com/rcdailey/cd3437bb2c63647126aa5740824b2a4f)

## Fixed behavior compared to `actions/upload-artifact`

*tbd*

1. Preserve file permissions
2. Defined root directory from where the content of the tarball is constructed.

## Limitations of `tar`

### On Linux

*tbd*


### On macOS

*tbd*


### On Windows

*tbd*


## Dependencies

* [actions/upload-artifact@v4](https://github.com/actions/upload-artifact)


## Contributors

* [Patrick Lehmann](https://GitHub.com/Paebbels) (Maintainer)
* [Sven Köhler](https://GitHub.com/skoehler)
* [and more...](https://GitHub.com/pyTooling/upload-artifact/graphs/contributors)
