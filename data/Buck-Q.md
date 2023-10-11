This test
https://github.com/code-423n4/2023-10-ens/blob/ed25379c06e42c8218eb1e80e141412496950685/test/delegatemulti.js#L182
does not actually test the functionality commented.  No tokens are ever transferred to 'secondDelegator' therefore delegation is not done to already delegated delegates.

I have updated the test here, to test this functionality.
https://github.com/craig-iam-smith/2023-10-ens/blob/main/test/delegatemulti.js
