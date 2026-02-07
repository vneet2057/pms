# Multi-Client User Management Implementation Summary

This document outlines all the changes made to support multi-client user management with invitation system and client switching functionality.

## Overview

The implementation allows:
1. Users to belong to multiple clients/workspaces
2. Inviting existing users to new clients without creating duplicate accounts
3. Email invitation links instead of sending passwords
4. Users accepting invitations to join new clients
5. Switching between different client workspaces from the dashboard

---

## Backend Changes

### 1. Database Model Updates

**File**: `backend/customer/models.py`

Added new fields to `UserClientRole` model:
- `invitation_status`: CharField with choices ('pending', 'accepted', 'rejected')
- `invited_at`: DateTimeField to track when invitation was sent
- `accepted_at`: DateTimeField to track when invitation was accepted

**Migration Required**: You need to run:
```bash
cd backend
python manage.py makemigrations customer
python manage.py migrate customer
```

### 2. User Management ViewSet Updates

**File**: `backend/user/adapters/viewsets/user_management_viewset.py`

#### Updated Methods:

**`create()` method**:
- Now checks if user already exists
- If user exists, adds them to the new client with 'pending' invitation status
- If user is new, creates account with 'pending' status
- Sends invitation email with token instead of password

**`bulk_create()` method**:
- Same logic as create() but for multiple users
- Handles existing users gracefully

#### New Endpoints Added:

1. **`accept_invitation/`** (POST) - Public endpoint
   - URL: `/user/team-members/accept_invitation/`
   - Accepts invitation token
   - Updates invitation_status to 'accepted'
   - Sets accepted_at timestamp
   - Updates user's active client

2. **`current_client/`** (GET) - Authenticated
   - URL: `/user/team-members/current_client/`
   - Returns the user's currently active client
   - Includes client id, name, schema_name, domain, role, icon
   - More efficient than fetching all clients when only current is needed

3. **`my_clients/`** (GET) - Authenticated
   - URL: `/user/team-members/my_clients/`
   - Returns all clients user belongs to
   - Shows which client is currently active
   - Includes client icon, name, role, etc.

4. **`switch_client/`** (POST) - Authenticated
   - URL: `/user/team-members/switch_client/`
   - Switches user's active client
   - Validates user has accepted invitation to that client
   - Updates ActiveClient table

### 3. Email Service Updates

**File**: `backend/utils/email_service.py`

**New Method**: `send_client_invitation_email()`
- Sends beautifully formatted HTML invitation emails
- Different templates for new vs existing users
- Includes invitation link instead of password
- Link format: `{FRONTEND_URL}/accept-invitation?token={token}`

**Token Format**: `{user_client_role.id}-{random_32_char_string}`

---

## Frontend Changes

### 1. New Type Definitions

**File**: `saas-pms-dashboard/src/types/client.ts`

Defines:
- `Client` interface - for listing all clients with is_active flag
- `CurrentClient` interface - for the current active client response
- `SwitchClientRequest` interface - request payload for switching clients
- `SwitchClientResponse` interface - response from switch operation

### 2. Client Service

**File**: `saas-pms-dashboard/src/services/ClientService.ts`

Three API methods:
- `fetchCurrentClient()`: Fetch the user's current active client (efficient for getting just current client info)
- `fetchMyClients()`: Fetch all user's clients with active status
- `switchActiveClient(clientId)`: Switch to a different client

### 3. Client Switcher Modal

**File**: `saas-pms-dashboard/src/components/ClientSwitcherModal.tsx`

Features:
- Shows all available workspaces
- Displays current active workspace
- Loading states with spinner
- Smooth transition when switching
- Shows client icons/avatars
- Displays user's role in each workspace
- Automatically reloads page after switching to apply new context

### 4. Updated Team Switcher

**File**: `saas-pms-dashboard/src/components/team-switcher.tsx`

Changes:
- Added chevron icon to indicate clickability
- Opens ClientSwitcherModal when clicked
- Shows current workspace information

### 5. Invitation Acceptance Page

**File**: `saas-pms-dashboard/src/pages/AcceptInvitation.tsx`

Features:
- Processes invitation token from URL query parameter
- Shows loading, success, or error states
- Redirects to dashboard on success
- Helpful error messages
- Beautiful UI with icons and cards

### 6. Routing Updates

**File**: `saas-pms-dashboard/src/App.tsx`

Added route:
```tsx
<Route path="/accept-invitation" element={<AcceptInvitation />} />
```

---

## User Flow

### Inviting a New User

1. Admin creates user in Settings > Team Members
2. System checks if email already exists
3. If new:
   - Creates user account with pending status
   - Sets this client as active client
4. If existing:
   - Adds user to UserClientRole with pending status
   - Does NOT change their active client yet
5. Sends invitation email with accept link

### Accepting Invitation

1. User clicks link in email
2. Frontend calls `/user/team-members/accept_invitation/`
3. Backend validates token
4. Updates invitation_status to 'accepted'
5. User can now access the workspace

### Switching Workspaces

1. User clicks on workspace name in sidebar
2. Modal shows all available workspaces
3. User selects different workspace
4. API updates ActiveClient table
5. Page reloads with new workspace context
6. Django-tenants middleware serves correct schema

---

## API Endpoints Summary

### User Management Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/user/team-members/` | Yes | Create/invite user |
| POST | `/user/team-members/bulk_create/` | Yes | Bulk create/invite users |
| GET | `/user/team-members/current_client/` | Yes | Get current active client |
| GET | `/user/team-members/my_clients/` | Yes | Get user's all clients |
| POST | `/user/team-members/switch_client/` | Yes | Switch active client |
| POST | `/user/team-members/accept_invitation/` | No | Accept invitation |

---

## Environment Variables

Add to your `.env`:

```bash
FRONTEND_URL=https://collabrix.hunchhadigital.com.np
```

This is used in invitation email links.

---

## Testing Checklist

### Backend Tests

- [ ] Create new user invitation
- [ ] Invite existing user to new client
- [ ] Accept invitation with valid token
- [ ] Reject invalid/expired token
- [ ] Switch between clients
- [ ] Verify ActiveClient updates correctly
- [ ] Check invitation_status transitions
- [ ] Test bulk user creation

### Frontend Tests

- [ ] Open client switcher modal
- [ ] Display all user's clients
- [ ] Switch to different workspace
- [ ] Accept invitation from email link
- [ ] Handle invalid invitation tokens
- [ ] Show loading states correctly
- [ ] Page reload after switching

### Email Tests

- [ ] New user receives invitation email
- [ ] Existing user receives invitation email
- [ ] Email contains correct invitation link
- [ ] Email formatting looks good

---

## Migration Steps

1. **Update Database Schema**:
   ```bash
   cd backend
   python manage.py makemigrations customer
   python manage.py migrate customer
   ```

2. **Update Existing Data** (Optional):
   If you have existing UserClientRole records, you may want to set their invitation_status to 'accepted':
   ```python
   # In Django shell
   from customer.models import UserClientRole
   UserClientRole.objects.all().update(invitation_status='accepted')
   ```

3. **Deploy Backend Changes**:
   - Deploy updated backend code
   - Run migrations
   - Restart server

4. **Deploy Frontend Changes**:
   - Build React app: `npm run build`
   - Deploy to hosting

---

## Known Limitations

1. **Token Security**: Tokens are currently simple (id + random string). Consider using JWT or signed tokens for production.
2. **Token Expiration**: Tokens don't expire. Consider adding expiration time.
3. **Email Rate Limiting**: No rate limiting on invitation emails.
4. **Invitation Revocation**: No way to revoke pending invitations yet.

---

## Future Enhancements

1. **Token Expiration**: Add expiration time to invitations (e.g., 7 days)
2. **Invitation Management**: UI to view/revoke pending invitations
3. **Email Templates**: More sophisticated email templates with branding
4. **Notification System**: In-app notifications for invitations
5. **Audit Log**: Track who invited whom and when
6. **Permission-based Invitations**: Restrict who can invite users
7. **SSO Integration**: Single sign-on for enterprise clients

---

## Support

For issues or questions:
1. Check Django logs for backend errors
2. Check browser console for frontend errors
3. Verify migrations have been applied
4. Ensure FRONTEND_URL environment variable is set correctly

---

**Implementation Date**: 2026-01-24
**Implemented By**: Claude Code Assistant
