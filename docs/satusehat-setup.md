# SatuSehat Integration Guide

## Overview

This guide explains how to connect SIMRS × CALL-E to Indonesia's [SatuSehat](https://satusehat.kemkes.go.id) national health data exchange platform.

## Prerequisites

1. **SatuSehat Developer Account** — Register at [satusehat.kemkes.go.id](https://satusehat.kemkes.go.id)
2. **Organization ID** — Provided after registration
3. **OAuth Credentials** — Client ID and Client Secret
4. **FHIR Resource Access** — Permission to create Appointment, Encounter, Communication resources

## Setup Steps

### Step 1: Register Your Organization

Go to [SatuSehat Developer Portal](https://satusehat.kemkes.go.id) and register your hospital. You'll receive:
- Organization ID (org_id)
- Client ID (client_id)
- Client Secret (client_secret)

### Step 2: Configure Environment

```bash
export SATUSEHAT_ORG_ID="your-org-id"
export SATUSEHAT_CLIENT_ID="your-client-id"
export SATUSEHAT_CLIENT_SECRET="your-client-secret"
export SATUSEHAT_BASE_URL="https://api-satusehat.dto.kemkes.go.id/fhir-r4/v1"
```

### Step 3: Get Access Token

```bash
curl -X POST https://api-satusehat.dto.kemkes.go.id/oauth2/v1/accesstoken \
  -d "client_id=$SATUSEHAT_CLIENT_ID" \
  -d "client_secret=$SATUSEHAT_CLIENT_SECRET" \
  -d "grant_type=client_credentials"
```

Response:
```json
{
  "access_token": "eyJhbGciOi...",
  "expires_in": 3600,
  "token_type": "bearer"
}
```

### Step 4: Create FHIR Appointment

After a patient confirms their appointment via CALL-E, update the FHIR Appointment:

```bash
curl -X PUT "$SATUSEHAT_BASE_URL/Appointment/APT-20260729-001" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/fhir+json" \
  -d '{
    "resourceType": "Appointment",
    "id": "APT-20260729-001",
    "status": "booked",
    "appointmentType": {
      "coding": [{
        "system": "http://terminology.hl7.org/CodeSystem/v2-0276",
        "code": "FOLLOWUP",
        "display": "A follow up visit"
      }]
    },
    "start": "2026-07-30T09:00:00+07:00",
    "end": "2026-07-30T09:30:00+07:00",
    "participant": [
      {
        "actor": {"reference": "Patient/patient-id"},
        "status": "accepted"
      },
      {
        "actor": {"reference": "Practitioner/pract-ahmad-ridwan"},
        "status": "accepted"
      }
    ]
  }'
```

### Step 5: Create FHIR Encounter (on arrival)

When the patient arrives for their appointment:

```bash
curl -X POST "$SATUSEHAT_BASE_URL/Encounter" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/fhir+json" \
  -d '{
    "resourceType": "Encounter",
    "status": "arrived",
    "class": {
      "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
      "code": "AMB",
      "display": "ambulatory"
    },
    "subject": {"reference": "Patient/patient-id"},
    "participant": [{
      "individual": {"reference": "Practitioner/pract-ahmad-ridwan"}
    }],
    "appointment": [{"reference": "Appointment/APT-20260729-001"}],
    "period": {
      "start": "2026-07-30T09:00:00+07:00"
    },
    "serviceProvider": {"reference": "Organization/$SATUSEHAT_ORG_ID"}
  }'
```

### Step 6: Log Communication (audit trail)

```bash
curl -X POST "$SATUSEHAT_BASE_URL/Communication" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/fhir+json" \
  -d '{
    "resourceType": "Communication",
    "status": "completed",
    "subject": {"reference": "Patient/patient-id"},
    "recipient": [{"reference": "Patient/patient-id"}],
    "payload": [{
      "contentString": "Appointment reminder call: confirmed attendance"
    }],
    "sent": "2026-07-29T15:30:00+07:00"
  }'
```

## Status Mapping

| CALL-E Outcome | FHIR Appointment Status |
|---|---|
| CONFIRMED | `booked` |
| RESCHEDULED | `cancelled` (old) + `booked` (new) |
| CANCELLED | `cancelled` |
| No-show | `noshow` |

## Troubleshooting

| Issue | Solution |
|---|---|
| 401 Unauthorized | Token expired — get new access token |
| 403 Forbidden | Check Organization ID and resource permissions |
| 404 Not Found | Resource ID doesn't exist — create first, then update |
| 422 Unprocessable | Check FHIR resource format — required fields missing |

## References

- [SatuSehat Documentation](https://satusehat.kemkes.go.id/platform)
- [FHIR R4 Specification](https://hl7.org/fhir/R4/)
- [Indonesian ICD-10 Codes](http://www.depkes.go.id/article/view/18090700003/klasifikasi-international-statistical-classification-of-diseases-and-related-health-problems-icd-10.html)
