# Contributing

Contributions to DCM are welcome. We follow the [Github flow](https://guides.github.com/introduction/flow/) development model for both [source code](https://github.com/Dronecode/camera-manager) and [documentation](https://github.com/Dronecode/camera-manager-docs).

Below we explain specific testing required by this project in order to submit changes, and also how to open a pull request.

## Pre-Submission Tests {#tests}

Before submitting PRs for source code changes, check [coding style](#coding_style), check for [memory leaks](#valgrind), and
[test](../test/README.md) on real hardware.

### Coding Style {#coding_style}

Check coding style using [tools/checkpatch](https://github.com/Dronecode/camera-manager/blob/master/tools/checkpatch):
```sh
./tools/checkpatch
```

### Memory Leak Testing (Valgrid) {#valgrind}

Check for memory leads using *valgrind*.

In order to avoid seeing a lot of *glib* and *gstreamer* memory leak false-positives, run *valgrind* using the following command:
```
GDEBUG=gc-friendly G_SLICE=always-malloc valgrind --suppressions=valgrind.supp --leak-check=full --track-origins=yes --show-possibly-lost=no --num-callers=20 ./dcm
```


## How to Open a Pull Request

1. First [fork and clone](https://help.github.com/articles/fork-a-repo) the project project.
1. Create a feature branch off master
   ```
   git checkout -b mydescriptivebranchname
   ```
   > **Note** *Always* branch off master for new features.
1. Commit your changes with a descriptive commit message.
   * Include context information, what was fixed, and an [issue number](https://github.com/Dronecode/camera-manager) (Github will link these then conveniently)
   * **Example:**

     ```
     Support new camera type

     - Fixes a typo
     - Adds specific xml config file

     Fixes issue #123
     ```

1. Test your changes [as described above](#tests) (we may ask you for test results in your PR).
1. Push changes to your repo:
   ```
   git push origin mydescriptivebranchname
   ```
1. Send a [pull request](https://github.com/Dronecode/camera-manager/compare/) to merge changes in the branch.
