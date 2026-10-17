# Restricted HR Folder (Folder Permissions)

## Ticket
Make an HR folder. The HR group can read, Managers can edit, and everyone else can't even open it.

## What I built
- Made an Organizational Unit called HR with two groups, `HR-Team` and `Managers`.
- Made two users: Pat Kim (`pkim`, in HR-Team) and Morgan Diaz (`mdiaz`, in Managers).
- Created the folder `C:\Shares\HR` and a test file `hr-test.txt` inside it.
- Removed the permissions the folder copied from its parent folder (`icacls /inheritance:r`), so nobody had access by default.
- Added back only what was needed: Administrators Full, HR-Team Read, Managers Modify.
- Shared the folder as `HR` with Everyone at Change, so the share is open and the folder permissions decide who actually gets in.

<img width="466" height="384" alt="VirtualBoxVM_WuztIg4ORr" src="https://github.com/user-attachments/assets/e43df160-2f22-49a8-800b-0705e9618d8a" />


<img width="466" height="384" alt="VirtualBoxVM_nnnubq56VJ" src="https://github.com/user-attachments/assets/bd659c2a-d56b-46a6-9d7d-332afe6ae4fe" />
## Test results (on CLIENT01)
| Account | Group | Result |
|---|---|---|
| Pat (`pkim`) | HR-Team | Could open the file, could not save changes |
| Morgan (`mdiaz`) | Managers | Could open, edit and save |
| `jit` | none | Access denied |

<img width="512" height="384" alt="VirtualBoxVM_1Ypwlg0g27" src="https://github.com/user-attachments/assets/714db685-2d74-4563-b9bc-df419cd5484f" />

<img width="512" height="384" alt="VirtualBoxVM_HTtoYKRBob" src="https://github.com/user-attachments/assets/c05e68a5-6e41-4ce4-8c99-424a6ee604f7" />
## Share permissions vs folder permissions
There are two locks. The share is the front door to the network, and the folder permissions are the lock on the cabinet. Both have to say yes, and when they disagree the stricter one wins. Here the share said Change for Everyone, but the folder only gave HR-Team Read, so Pat got Read.

## Why someone can be denied even in the right group
- The share and the folder each have their own permissions, and the stricter one wins.
- A Deny rule beats any Allow.
- Group membership only updates when the user signs in, so someone added to a group has to sign out and back in.
