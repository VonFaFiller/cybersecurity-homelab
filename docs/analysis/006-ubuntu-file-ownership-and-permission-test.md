# Ubuntu File Ownership and Permission Test

## Objective
Verify how file ownership and file permissions interact when access is attempted by different local users.

## Why this matters
This exercise helps distinguish between file ownership and file permissions.
It shows that access control depends not only on the permission bits of a file, but also on which user owns it.

## Observations
- A file was created as `testuser`.
- The file owner was initially `testuser`.
- After setting the file mode to `600`, access was restricted to the owner only.
- The normal user could not read the file while `testuser` remained the owner.
- After changing the owner of the file with `chown`, the normal user was able to read it.
- The exercise showed that ownership and permission bits both influence access control.

## Result
The exercise confirmed that file permissions alone do not fully determine access.
Ownership is also a key factor, and changing the owner of a file can change who is able to access it.
