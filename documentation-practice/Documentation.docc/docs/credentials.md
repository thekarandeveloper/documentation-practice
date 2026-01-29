# Credentials

This document contains all credentials and API keys for the iOS Swipe project.

> **SECURITY WARNING**: This file contains sensitive information. Ensure this file is:
> - Added to `.gitignore` to prevent committing to version control
> - Stored securely with appropriate access controls
> - Never shared publicly or in unsecured channels

---

## Table of Contents
- [API Credentials](#api-credentials)
- [Firebase Configuration](#firebase-configuration)
- [Database Credentials](#database-credentials)
- [Third-Party Services](#third-party-services)
- [Testing & Development Tools](#testing--development-tools)
- [Environment Variables](#environment-variables)
- [App Store Connect](#app-store-connect)
- [Code Signing](#code-signing)
- [Resources & Documentation](#resources--documentation)

---

## API Credentials

### Backend API
- **Base URL (Development)**: `https://dev.getswipe.in`
- **Base URL (Staging)**:
- **Base URL (Production)**: `https://app.getswipe.in`
- **API Key**:
- **API Secret**:

### Customer Device Access

#### Read Only Access
**Endpoint**: `POST https://app.getswipe.in/api/user/login`

**Request Body**:
```json
{
  "email": 9915591138,
  "password": 513531535613513
}
```
**Note**: Add the returned token to Shared Preferences

#### Full Access (Not Read Only)
**Endpoint**: `POST http://app.getswipe.in/api/utils/app_token/8388f5c3-d25d-4fb5-848c-e2437729cd46`

**Request Body**:
```json
{
  "pass_code": "Android@404",
  "company_id": <ADD COMPANY ID>
}
```

### Document Calculation Logs
**Endpoint**: `GET https://dev.getswipe.in/api/utils/get_android_user_trace/{session_id}`

**Description**: Get calculation error logs. Session ID is received in the create bill payload shared in Calculation Error mail.

**Response**: Returns `.txt` file with array of logs

### Authentication
- **OAuth Client ID**:
- **OAuth Client Secret**:
- **OAuth Redirect URI**:

---

## Firebase Configuration

### iOS App
- **Project ID**:
- **App ID**:
- **API Key**:
- **Sender ID**:
- **Storage Bucket**:
- **GoogleService-Info.plist**: (Location: `Swipe/GoogleService-Info.plist`)

### Firebase Services
- **Analytics**: Enabled/Disabled
- **Crashlytics**: Enabled/Disabled
- **Cloud Messaging (FCM)**:
  - **Server Key**:
  - **Sender ID**:
- **Remote Config**: Enabled/Disabled

---

## Database Credentials

### Realm Database
- **App ID**:
- **Encryption Key**:
- **Sync URL**:

### Backend Database (if applicable)
- **Host**:
- **Port**:
- **Database Name**:
- **Username**:
- **Password**:

---

## Third-Party Services

### Payment Gateway
- **Provider**:
- **Merchant ID**:
- **API Key**:
- **Secret Key**:
- **Webhook Secret**:

### Analytics Services
- **Microsoft Clarity**:
  - **Project ID**:
  - **API Key**:

- **Mixpanel/Amplitude** (if applicable):
  - **Project Token**:
  - **API Secret**:

### Push Notifications
- **APNs Authentication Key ID**:
- **APNs Team ID**:
- **APNs Key File**: (Location: )
- **APNs Key (.p8)**:

### SMS/OTP Service
- **Provider**:
- **Account SID**:
- **Auth Token**:
- **Phone Number**:

### Cloud Storage
- **Provider** (AWS S3/GCS/etc.):
- **Bucket Name**:
- **Access Key ID**:
- **Secret Access Key**:
- **Region**:

---

## Testing & Development Tools

### BrowserStack
- **Email**: `backend@getswipe.in`
- **Password**: `swipe123#`

### Postman
- **Email**: `backend@getswipe.in`
- **Password**: `Swipeman700`

### Font Awesome
- **Username**: `team@getswipe.in`
- **Password**: `getswipe.in`

---

## Environment Variables

### Development
```
API_BASE_URL=
API_KEY=
ENVIRONMENT=development
DEBUG_MODE=true
```

### Staging
```
API_BASE_URL=
API_KEY=
ENVIRONMENT=staging
DEBUG_MODE=true
```

### Production
```
API_BASE_URL=
API_KEY=
ENVIRONMENT=production
DEBUG_MODE=false
```

---

## App Store Connect

### Account Details
- **Apple ID**:
- **Team ID**:
- **Team Name**:

### App Information
- **App ID**:
- **Bundle Identifier**: `com.swipe.mac`
- **SKU**:

### App Store Connect API
- **Key ID**: `53X848C4MG`
- **Issuer ID**: `1d03a2af-2a6a-4127-acd9-fa4d12d7b6d8`
- **Private Key (.p8)**: (Location: )

### In-App Purchases
- **Shared Secret**:

---

## Code Signing

### Development
- **Certificate**:
- **Provisioning Profile**:
- **Team ID**:

### Distribution
- **Certificate**:
- **Provisioning Profile**:
- **Team ID**:

### Keychain Access
- **Keychain Password** (for CI/CD):

---

## Resources & Documentation

### Testing Resources
- **Swipe Test Checklist**: https://docs.google.com/spreadsheets/d/1x3rxF8vaCw95LlOOqcAVgP-m-QPHVELWL_8tcRC0lnc/edit?gid=0#gid=0

### Analytics & Tracking
- **iOS Analytics Tracking Sheet**: https://docs.google.com/document/d/1DePnvNTlcnfrJCc7QQTfQ2LuNUthsEL7Va901WGFd-U/edit?tab=t.0#heading=h.im205jsbxx4i

---

**Last Updated**: 2025-11-22
