# OpenDeskPro v2.0 - Folder Structure & Tool Features Documentation

## 📁 Project Folder Structure

### Root Level
```
opendeskpro-v2.0/
├── backend/              # Python FastAPI backend (alternative backend)
├── frontend/             # React frontend application
├── server/               # Node.js/Express backend (main server)
├── dist-release/         # Distribution/build files
├── installation_files/   # Installation scripts and configs
├── scripts/              # Build and utility scripts
├── License/              # License files for Pro activation
└── ssl/                  # SSL certificates
```

### Key Directories

#### **Frontend** (`frontend/`)
- **`src/components/`** - React components (UI, layout, widgets)
- **`src/pages/`** - Page components (Admin, Tickets, Settings, etc.)
- **`src/hooks/`** - Custom React hooks (including `useFeatureAccess`)
- **`src/contexts/`** - React contexts (Auth, Theme, Logo, SSO)
- **`src/services/`** - API and service integrations
- **`src/utils/`** - Utility functions

#### **Server** (`server/`)
- **`routes/`** - API route handlers
- **`models/`** - MongoDB/Mongoose data models
- **`middleware/`** - Express middleware (auth, feature checks, uploads)
- **`services/`** - Business logic services
- **`workers/`** - Background workers (email, SLA, automation)
- **`config/`** - Configuration files
- **`scripts/`** - Server-side utility scripts

---

## 🔐 Tool Features System

### Overview
The Tool Features system is a **freemium feature gating mechanism** that controls access to premium (PRO) features based on:
1. Organization subscription plan (BASIC vs PRO)
2. License file activation (file-based licensing)
3. Lock file activation (destructive activation method)

### Feature List

#### **PRO Features** (Require PRO Plan)
All these features are gated behind PRO plan access:

1. **`SLA_MANAGER`** - Service Level Agreement management
2. **`SSO_INTEGRATION`** - Single Sign-On configuration
3. **`EXTERNAL_INTEGRATIONS`** - External API integrations and webhooks
4. **`ADVANCED_REPORTS`** - Advanced reporting and analytics
5. **`EMAIL_AUTOMATION`** - Email automation workflows
6. **`TEAMS_INTEGRATION`** - Microsoft Teams integration
7. **`AZURE_SENTINEL`** - Azure Sentinel integration
8. **`DOMAIN_RULES`** - Domain whitelist/blacklist rules
9. **`CUSTOM_ROLES`** - Custom role management

#### **Basic Features** (Available to all users)
- Ticket Management
- Email Integration (SMTP/IMAP)
- User & Department Management
- Basic Reports
- API Keys
- Email Templates
- FAQ Management
- Chatbot

---

## 🏗️ Architecture

### Backend Implementation

#### **1. Feature Middleware** (`server/middleware/checkFeature.js`)
```javascript
// Usage in routes
router.use(protect, admin, checkFeature('TEAMS_INTEGRATION'))
```

**How it works:**
- Factory function that creates Express middleware
- Checks feature access in priority order:
  1. **Lock file** (destructive activation - highest priority)
  2. **License file** (`License/license.json` - image-based activation)
  3. **Database plan** (Organization.plan === 'PRO' + subscription expiry check)

**Key Functions:**
- `checkFeature(featureName)` - Middleware factory
- `hasFeatureAccess(organization, featureName)` - Helper function

#### **2. License Service** (`server/services/licenseService.js`)
- Validates `License/license.json` file
- Checks for lock file via `lockFileService`
- Provides `isProEnabled()` function
- In-memory license state caching (5-minute refresh)

#### **3. Organization Model** (`server/models/Organization.js`)
```javascript
{
  plan: 'BASIC' | 'PRO',
  subscriptionExpiry: Date,
  paymentStatus: 'PENDING' | 'VERIFIED' | 'FAILED'
}
```

### Frontend Implementation

#### **1. Feature Access Hook** (`frontend/src/hooks/useFeatureAccess.js`)
```javascript
const { hasAccess, plan, isPro, isBasic } = useFeatureAccess()

// Check feature access
if (hasAccess('SLA_MANAGER')) {
  // Show SLA features
}
```

**Returns:**
- `hasAccess(featureName)` - Function to check feature access
- `plan` - Current plan ('BASIC' or 'PRO')
- `isPro` - Boolean indicating PRO status
- `isBasic` - Boolean indicating BASIC status
- `subscriptionExpiry` - Subscription expiry date

#### **2. Sidebar Integration** (`frontend/src/components/layout/Sidebar.jsx`)
- Menu items have `feature` property
- Locked features show lock icon
- Clicking locked features shows upgrade modal
- Features are conditionally rendered based on access

**Example:**
```javascript
{ path: '/admin/sla', icon: Clock, label: 'SLA Policies', feature: 'SLA_MANAGER' }
```

---

## 📍 Feature Usage Locations

### Backend Routes Using Features

1. **`server/routes/teams.js`**
   - Feature: `TEAMS_INTEGRATION`
   - Usage: `router.use(checkFeature('TEAMS_INTEGRATION'))`

2. **`server/routes/emailAutomation.js`**
   - Feature: `EMAIL_AUTOMATION`
   - Usage: `router.use(checkFeature('EMAIL_AUTOMATION'))`

3. **`server/routes/integrations.js`**
   - Feature: `EXTERNAL_INTEGRATIONS`
   - Usage: `router.use(checkFeature('EXTERNAL_INTEGRATIONS'))`

4. **`server/routes/reports.js`**
   - Feature: `ADVANCED_REPORTS`
   - Usage: Applied to specific routes (dashboard stats)

5. **`server/routes/admin.js`**
   - Features: `DOMAIN_RULES`, `CUSTOM_ROLES`
   - Usage: Inline checks for domain rules endpoint

### Frontend Pages Using Features

1. **Admin Pages:**
   - `Admin/SLA.jsx` - `SLA_MANAGER`
   - `Admin/SSOConfig.jsx` - `SSO_INTEGRATION`
   - `Admin/Integrations.jsx` - `EXTERNAL_INTEGRATIONS`
   - `Admin/EmailAutomation.jsx` - `EMAIL_AUTOMATION`
   - `Admin/DomainRules.jsx` - `DOMAIN_RULES`
   - `Admin/Roles.jsx` - `CUSTOM_ROLES`
   - `Admin/Analytics.jsx` - `ADVANCED_REPORTS`

2. **Upgrade Pages:**
   - `Upgrade/UpgradePlan.jsx` - Shows feature comparison
   - `ProActivation.jsx` - License activation interface

---

## 🔄 Feature Check Flow

### Backend Flow
```
Request → Auth Middleware → checkFeature Middleware
                                    ↓
                    Check Lock File (Priority 1)
                                    ↓ (if not found)
                    Check License File (Priority 2)
                                    ↓ (if not found)
                    Check Database Plan (Priority 3)
                                    ↓
                    Allow/Deny Request
```

### Frontend Flow
```
Component → useFeatureAccess Hook
                    ↓
        Check user.organization.plan
                    ↓
        Check subscriptionExpiry
                    ↓
        Return hasAccess function
                    ↓
        Conditionally render UI
```

---

## 📝 Feature Constants

### Backend (`server/middleware/checkFeature.js`)
```javascript
const FEATURE_PLAN_REQUIREMENTS = {
  SLA_MANAGER: 'PRO',
  SSO_INTEGRATION: 'PRO',
  EXTERNAL_INTEGRATIONS: 'PRO',
  ADVANCED_REPORTS: 'PRO',
  EMAIL_AUTOMATION: 'PRO',
  TEAMS_INTEGRATION: 'PRO',
  AZURE_SENTINEL: 'PRO',
  DOMAIN_RULES: 'PRO',
  CUSTOM_ROLES: 'PRO',
}
```

### Frontend (`frontend/src/hooks/useFeatureAccess.js`)
```javascript
const FEATURE_PLAN_REQUIREMENTS = {
  SLA_MANAGER: 'PRO',
  SSO_INTEGRATION: 'PRO',
  EXTERNAL_INTEGRATIONS: 'PRO',
  ADVANCED_REPORTS: 'PRO',
  EMAIL_AUTOMATION: 'PRO',
  TEAMS_INTEGRATION: 'PRO',
  AZURE_SENTINEL: 'PRO',
  DOMAIN_RULES: 'PRO',
  CUSTOM_ROLES: 'PRO',
}
```

**⚠️ Note:** These constants must be kept in sync between frontend and backend!

---

## 🎯 Adding a New Feature

### Step 1: Add Feature Constant
Add to both:
- `server/middleware/checkFeature.js`
- `frontend/src/hooks/useFeatureAccess.js`

```javascript
const FEATURE_PLAN_REQUIREMENTS = {
  // ... existing features
  NEW_FEATURE: 'PRO',  // or 'BASIC' if available to all
}
```

### Step 2: Protect Backend Route
```javascript
import { checkFeature } from '../middleware/checkFeature.js'

router.use(protect, admin, checkFeature('NEW_FEATURE'))
```

### Step 3: Protect Frontend Component
```javascript
import { useFeatureAccess } from '../../hooks/useFeatureAccess'

const { hasAccess } = useFeatureAccess()

if (!hasAccess('NEW_FEATURE')) {
  return <UpgradePrompt />
}
```

### Step 4: Update Sidebar (if needed)
```javascript
{ 
  path: '/admin/new-feature', 
  icon: Icon, 
  label: 'New Feature', 
  feature: 'NEW_FEATURE' 
}
```

### Step 5: Update Feature Lists
- `frontend/src/pages/Upgrade/UpgradePlan.jsx` - Add to feature comparison
- Update any documentation

---

## 🔍 Key Files Reference

| File | Purpose |
|------|---------|
| `server/middleware/checkFeature.js` | Backend feature gating middleware |
| `frontend/src/hooks/useFeatureAccess.js` | Frontend feature access hook |
| `server/services/licenseService.js` | License file validation |
| `server/services/lockFileService.js` | Lock file checking |
| `server/models/Organization.js` | Organization plan model |
| `frontend/src/components/layout/Sidebar.jsx` | Navigation with feature gating |
| `frontend/src/pages/Upgrade/UpgradePlan.jsx` | Feature comparison UI |

---

## 🚀 License Activation Methods

### Method 1: Lock File (Destructive Activation)
- Highest priority
- Permanent activation
- Checked via `lockFileService.checkLockFile()`

### Method 2: License File (Image-Based)
- Second priority
- File: `License/license.json`
- Validated via `licenseService.isProEnabled()`
- Supports PRO and PRO_LIFETIME tiers

### Method 3: Database Plan (Subscription)
- Fallback method
- Requires `Organization.plan === 'PRO'`
- Checks `subscriptionExpiry` date
- Used for subscription-based access

---

## 📊 Feature Access Matrix

| Feature | BASIC Plan | PRO Plan |
|---------|-----------|----------|
| Ticket Management | ✅ | ✅ |
| Email Integration | ✅ | ✅ |
| User Management | ✅ | ✅ |
| Basic Reports | ✅ | ✅ |
| API Keys | ✅ | ✅ |
| Email Templates | ✅ | ✅ |
| FAQ Management | ✅ | ✅ |
| Chatbot | ✅ | ✅ |
| SLA Management | ❌ | ✅ |
| SSO Integration | ❌ | ✅ |
| External Integrations | ❌ | ✅ |
| Advanced Reports | ❌ | ✅ |
| Email Automation | ❌ | ✅ |
| Domain Rules | ❌ | ✅ |
| Custom Roles | ❌ | ✅ |
| Teams Integration | ❌ | ✅ |
| Azure Sentinel | ❌ | ✅ |

---

## 🔧 Troubleshooting

### Feature Not Working
1. Check license file exists: `License/license.json`
2. Verify lock file (if using destructive activation)
3. Check organization plan in database
4. Verify subscription expiry date
5. Check feature constant spelling matches in both frontend and backend

### Frontend Shows Locked Feature
- Verify `useFeatureAccess` hook is being used
- Check user's organization is populated
- Verify plan is 'PRO' in database
- Check subscription hasn't expired

### Backend Returns 403
- Verify `checkFeature` middleware is applied
- Check license service is working
- Verify organization plan in database
- Check middleware order (auth before feature check)

---

## 📚 Related Documentation

- License System: See `server/services/licenseService.js`
- Lock File System: See `server/services/lockFileService.js`
- Organization Model: See `server/models/Organization.js`
- Upgrade Flow: See `frontend/src/pages/Upgrade/UpgradePlan.jsx`

