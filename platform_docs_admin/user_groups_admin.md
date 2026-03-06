# User Groups

## Overview

The platform provides a group system for organising users. Groups can be **user-created** (custom) or **system-managed** (automatically maintained by the platform).

---

## System Groups

The platform automatically maintains two system groups:

| Group   | Description | Membership Rule |
|---------|-------------|-----------------|
| **Global** | Every active user is automatically a member. | All users |
| **Admin**  | Every user with an admin or system admin role is automatically a member. | Admin-role users only |

### Protections

System groups are protected from accidental modification:

- **Cannot be renamed** — The name is fixed by the platform.
- **Cannot be deleted** — System groups are permanent.
- **Cannot have members manually removed** — Membership is managed automatically. If you try to remove a member from a system group, the platform will block the action.

### Automatic Reconciliation

The platform automatically keeps system group membership accurate:

1. On every server start, both system groups are verified and any missing members are added.
2. For the Admin group, users who no longer hold an admin role are automatically removed.
3. This guarantees correctness even if a prior update was missed.

### Using System Groups with Document Spaces

System groups can be granted access to any organizational document space, just like custom groups. For example:

- Grant the **Global** group **Read** access to a space to make it visible to all users
- Grant the **Admin** group **Write** access to a space to let all admins contribute

Tenant admins configure these permissions in the Document Management page under Space Management.

---

## Admin Group Sync

Admin group membership is kept in sync automatically at three points:

| Trigger | What Happens |
|---------|--------------|
| **Server start** | Full reconciliation — adds missing admins, removes users who are no longer admins |
| **User creation** | If the new user has an admin role, they are added to the Admin group |
| **Role change** | When a user's role is changed (e.g., promoted to admin or demoted), their Admin group membership is updated accordingly |

### Which roles are considered admin-level?

Users with either the **Admin** or **System Admin** role are automatically added to the Admin group.

---

## Custom Groups

Users can create custom groups to organise their teams. Custom groups support:

- Free-form name and description.
- Members with a role of **Member** or **Manager**.
- Integration with document space permissions — grant a group access to an organizational space and all its members automatically gain that access.

Custom groups have no automatic membership — members are added and removed manually.

---

## Managing Groups

### Creating a Group

1. Navigate to the groups area
2. Click **Create Group**
3. Enter a name and optional description
4. Click **Create**

### Adding Members

1. Open the group
2. Click **Add Member**
3. Select the user and choose a role (Member or Manager)
4. Click **Add**

### Removing Members

1. Open the group
2. Find the member in the list
3. Click the remove button next to their name

> **Note:** You cannot remove members from system groups (Global or Admin). The platform manages their membership automatically.

### Editing or Deleting a Group

- Click **Edit** on a group to change its name or description.
- Click **Delete** to soft-delete the group. System groups cannot be deleted.
