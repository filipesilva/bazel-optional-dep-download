# bazel-optional-dep-download

This package shows how optional dependencies are still downloaded even though they are known to be incompatible with the current platform. 
This example used Node 10.10.0, NPM 6.4.1 and Yarn 1.9.3.

Ignoring incompatible optional packages is desirable because helps reduce the total download size.

To reproduce the issue, run `npm run test-npm` or `yarn test-yarn`. 
This installs `@bazel/bazel`, which contains 3 optional dependencies with OS restrictions :
- `@bazel/bazel-linux_x64` containing `"os": ["linux"]`
- `@bazel/bazel-darwin_x64` containing `"os":["linux"]`
- `@bazel/bazel-win32_x64` containing `"os": ["win32"]`

Each of these platform specific packages contains a large (~160mb) binary.

The final installed node_modules contains only one of these packages, but through the verbose logs we can see that all three were downloaded:

- `npm`
```
npm http fetch GET 200 https://registry.npmjs.org/@bazel%2fbazel-linux_x64 1383ms
npm http fetch GET 200 https://registry.npmjs.org/@bazel%2fbazel-darwin_x64 1427ms
npm http fetch GET 200 https://registry.npmjs.org/@bazel%2fbazel-win32_x64 1439ms
// ...
npm http fetch GET 200 https://registry.npmjs.org/@bazel/bazel-linux_x64/-/bazel-linux_x64-0.18.0-rc4.tgz 6038ms
npm http fetch GET 200 https://registry.npmjs.org/@bazel/bazel-win32_x64/-/bazel-win32_x64-0.18.0-rc4.tgz 21486ms
npm http fetch GET 200 https://registry.npmjs.org/@bazel/bazel-darwin_x64/-/bazel-darwin_x64-0.18.0-rc4.tgz 25509ms
```

- `yarn`
```
verbose 1.347 Performing "GET" request to "https://registry.yarnpkg.com/@bazel%2fbazel-linux_x64".
verbose 1.348 Performing "GET" request to "https://registry.yarnpkg.com/@bazel%2fbazel-darwin_x64".
verbose 1.349 Performing "GET" request to "https://registry.yarnpkg.com/@bazel%2fbazel-win32_x64".
// ...
verbose 2.152 Performing "GET" request to "https://registry.yarnpkg.com/@bazel/bazel-linux_x64/-/bazel-linux_x64-0.18.0-rc4.tgz".
verbose 2.156 Performing "GET" request to "https://registry.yarnpkg.com/@bazel/bazel-win32_x64/-/bazel-win32_x64-0.18.0-rc4.tgz".
verbose 2.163 Performing "GET" request to "https://registry.yarnpkg.com/@bazel/bazel-darwin_x64/-/bazel-darwin_x64-0.18.0-rc4.tgz".
```

The first set of requests already contains the package.json data for the packages so it is already possible to know that the package is incompatible with the OS, and thus should be ignored since it is a optional. 
There should be no need to download it.

