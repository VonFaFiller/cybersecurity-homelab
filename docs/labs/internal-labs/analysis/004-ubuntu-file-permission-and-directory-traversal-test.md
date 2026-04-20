# Ubuntu File Permission and Directory Traversal Test

## Objective
Verify how file permissions and directory traversal permissions affect access to a file from another local user account.

## Why this matters
This exercise shows that file access depends not only on the permissions of the file itself, but also on the permissions of the directories in its path. It helps distinguish between file readability and path accessibility.

## Observations
- A file named `secret.txt` was created inside `/home/lab-username/labfiles/`.
- With permission mode `600`, the file was readable only by the owner.
- After changing the file to `644`, `testuser` still could not read it.
- After allowing directory traversal on the required path, `testuser` was able to read the file.
- Restoring restrictive permissions removed that access again.

## Issue Encountered
- File read access still failed after changing the file permissions because the parent directory did not allow traversal.

## Result
The exercise confirmed that successful file access depends on both file permissions and directory traversal permissions. A readable file is not enough if the path leading to it is not accessible to the requesting user.
