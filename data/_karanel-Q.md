## Unsynchronized lock files `package-lock.json` and `yarn.lock`

Severity: `Non-critical (QA)`

The `package-lock.json` and `yarn.lock` files are unsynchronized.
In the `package-lock.json` file, we can see that the version of `@openzeppelin/contracts` used is `v4.3.2`
https://github.com/code-423n4/2023-10-ens/blob/main/package-lock.json#L780-L784
```
    "@openzeppelin/contracts": {
      "version": "4.3.2",
      "resolved": "https://registry.npmjs.org/@openzeppelin/contracts/-/contracts-4.3.2.tgz",
      "integrity": "sha512-AybF1cesONZStg5kWf6ao9OlqTZuPqddvprc0ky7lrUVOjXeKpmQ2Y9FK+6ygxasb+4aic4O5pneFBfwVsRRRg=="
    },
```

Meanwhile, in the `yarn.lock` file, the version of `@openzeppelin/contracts` used is `v4.7.2`
https://github.com/code-423n4/2023-10-ens/blob/main/yarn.lock#L1199-L1202
```
"@openzeppelin/contracts@^4.0.0", "@openzeppelin/contracts@^4.1.0", "@openzeppelin/contracts@^4.3.1":
  version "4.7.2"
  resolved "https://registry.yarnpkg.com/@openzeppelin/contracts/-/contracts-4.7.2.tgz#7587416fe2d35abf574193515b8971bfe9f64bc7"
  integrity sha512-4n/JL9izql8303mPqPdubuna/DWEMbmOzWYUWyCPhjhiEr2w3nQrjE7vZz1fBF+wzzP6dZbIcsgqACk53c9FGA==
```


At first look, this doesn't seem like a big issue. But if a user/developer uses `npm` instead of `yarn` to install the dependencies and run the test in `test/delegatemulti.js` file, 4 out of 16 tests would fail.

This is because OpenZeppelin made drastic changes to their smart contracts from `v4.3.2` to `v4.7.2`, and the inconsistent versions in `package-lock.json` and `yarn.lock` files would cause 4 tests to fail.

### Error (12 passing, 4 failing):
```
ENS Multi Delegate
  deposit
    ✔ should be able to delegate multiple delegates in behalf of user
    ✔ should be able to delegate to already delegated delegates
    ✔ should revert if no source and target provided
  re-deposit
    ✔ should be able to re-delegate multiple delegates in behalf of user (1:1)
    ✔ should be able to re-delegate multiple delegates in behalf of user (many:many)
    ✔ should be able to re-delegate to already delegated delegates

    1) should revert if target amount is higher than source amount + caller allowance
    ✔ should revert if at least one source address is not delegate of the caller
  withdraw
    ✔ should be able to withdraw fully
    ✔ should be able to withdraw partially
    ✔ should fail to withdraw if amount exceeds

    2) should fail to withdraw if delegate was not delegated
  allowance

    3) should revert if allowance is not provided

    4) should revert if allowance is lesser than provided amount
  metadata uri
    ✔ deployer should be able to update metadata uri
    ✔ others should not be able to update metadata uri
```

### Suggested Mitigation

Any of the following would be a fix:
- Delete `package-lock.json` file
- Recreate and synchronize `package-lock.json` file in accordance with `yarn.lock` file.
- Change the version of `@openzeppelin/contracts` to `^4.6.0` in `package.json` file.