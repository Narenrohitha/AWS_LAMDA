<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=FF6B00&height=200&section=header&text=AWS%20Serverless%20Website&fontSize=42&fontColor=ffffff&fontAlignY=38&desc=Production-Grade%20%7C%20Zero%20Servers%20%7C%20Under%20%248%2Fmonth&descAlignY=58&descSize=16" width="100%"/>

<br/>

[![AWS](https://img.shields.io/badge/AWS-Serverless-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![CloudFront](https://img.shields.io/badge/CloudFront-CDN-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudfront)
[![License](https://img.shields.io/badge/License-MIT-22C55E?style=for-the-badge)](LICENSE)
[![Cost](https://img.shields.io/badge/Monthly%20Cost-%246--8-FF6B00?style=for-the-badge&logo=amazonwebservices&logoColor=white)](https://aws.amazon.com/pricing)

<br/>

> **No EC2. No servers to manage. No 3 AM "disk is full" nightmares.**
> Pure serverless, auto-scaling AWS infrastructure that costs almost nothing.

<br/>

**[🚀 Live Demo](https://rrlearn.xyz)** • **[📖 Full Article on Medium](https://medium.com/@narengl2001)** • **[👤 Author](https://linkedin.com/in/naren-g-7bb580229)**

</div>

---

## 📐 Architecture Overview

```
                           ┌─────────────────────────────────────────┐
                           │           rrlearn.xyz (HTTPS)           │
                           └─────────────────┬───────────────────────┘
                                             │
                                      ┌──────▼──────┐
                                      │  Route 53   │  DNS Resolution
                                      └──────┬──────┘
                                             │
                                      ┌──────▼──────┐
                              ┌───────│  CloudFront │───────┐
                              │       │  + WAF 🛡️   │       │
                              │       └─────────────┘       │
                              │                             │
                       ┌──────▼──────┐               ┌──────▼──────┐
                       │  S3 Bucket  │               │ API Gateway │
                       │  (Website)  │               │  (REST API) │
                       │  HTML/CSS/JS│               └──────┬──────┘
                       └─────────────┘                      │
                                              ┌─────────────┼─────────────┐
                                              │             │             │
                                       ┌──────▼───┐  ┌──────▼───┐  ┌──────▼───┐
                                       │ contact- │  │ upload-  │  │  api-    │
                                       │ handler  │  │ handler  │  │ handler  │
                                       └──────┬───┘  └──────┬───┘  └──────┬───┘
                                              │             │             │
                                       ┌──────▼───┐  ┌──────▼───┐  ┌──────▼───┐
                                       │DynamoDB  │  │S3 Uploads│  │DynamoDB  │
                                       │contacts  │  │(presigned│  │api-data  │
                                       └──────┬───┘  │  URLs)   │  └──────────┘
                                              │      └──────────┘
                                       ┌──────▼───┐
                                       │   SES    │  📧 Email
                                       └──────────┘
```

---

## ✨ What's Inside

| Layer | Service | Purpose |
|:------|:--------|:--------|
| 🌐 **DNS** | Route 53 | Routes `rrlearn.xyz` → CloudFront IP |
| 🔒 **CDN + TLS** | CloudFront + ACM | HTTPS everywhere, global edge caching |
| 🛡️ **Security** | WAF | Blocks SQLi, XSS, bad IPs, rate limits |
| 📦 **Static Files** | S3 (website bucket) | HTML/CSS/JS — private, OAC access only |
| 📤 **File Uploads** | S3 (uploads bucket) | User files via presigned URLs |
| 🔌 **API** | API Gateway | Routes `/contact` `/upload` `/api/*` |
| ⚡ **Compute** | Lambda × 3 | contact-handler · upload-handler · api-handler |
| 🗄️ **Database** | DynamoDB | `contacts` table + `api-data` table |
| 📧 **Email** | SES | Contact form notifications |
| 🔑 **Auth** | IAM Role | Least-privilege Lambda permissions |
| 🔐 **SSL** | ACM | Free certificate, auto-renews forever |

---

## 🗂️ Repository Structure

```
aws-serverless-website/
│
├── 📁 lambda/
│   ├── contact_handler.py       # Contact form → DynamoDB + SES
│   ├── upload_handler.py        # Presigned S3 URL generator
│   └── api_handler.py           # Full CRUD REST API
│
├── 📁 frontend/
│   ├── index.html
│   └── assets/
│       ├── css/
│       │   └── style.css
│       └── js/
│           ├── api.js           # ← Set API_BASE = 'https://rrlearn.xyz'
│           └── app.js
│
├── 📁 policies/
│   ├── uploads-bucket-policy.json
│   ├── uploads-bucket-cors.json
│   └── lambda-iam-role.json
│
├── 📁 docs/
│   └── architecture.md
│
└── README.md
```

---

## ⚡ Quick Deploy — Build Order

> **Follow this exact order.** Skipping ahead causes dependency failures.

```
1️⃣  S3 Buckets       →  2️⃣  DynamoDB Tables   →  3️⃣  IAM Role
        ↓
4️⃣  Lambda × 3      →  5️⃣  API Gateway       →  6️⃣  CloudFront
        ↓
7️⃣  Bucket Policies  →  8️⃣  Route 53          →  9️⃣  SSL + Domain
        ↓
🔟  SES Email        →  1️⃣1️⃣  WAF              →  1️⃣2️⃣  Upload Frontend
        ↓
1️⃣3️⃣  Test Everything ✅
```

---

## 🪣 Step 1 — S3 Buckets

Create **two buckets** with different security postures:

<table>
<tr>
<td>

**Bucket 1 — Website Files**

| Setting | Value |
|---------|-------|
| Name | `mybusiness-website-prod1` |
| Region | `us-east-1` |
| Block public access | ✅ ON |
| Versioning | Enable |
| Encryption | SSE-S3 |

</td>
<td>

**Bucket 2 — User Uploads**

| Setting | Value |
|---------|-------|
| Name | `mybusiness-uploads-prod1` |
| Region | `us-east-1` |
| Block public access | ❌ OFF |
| Versioning | Enable |
| Encryption | SSE-S3 |

</td>
</tr>
</table>

> ⚠️ **Don't forget:** Add the uploads bucket policy in Step 7b, or presigned uploads will fail with `403`.

---

## 🗃️ Step 2 — DynamoDB Tables

<table>
<tr>
<td>

**Table 1 — contacts**

| Setting | Value |
|---------|-------|
| Table name | `contacts` |
| Partition key | `id` (String) |
| Billing mode | On-demand |

</td>
<td>

**Table 2 — api-data**

| Setting | Value |
|---------|-------|
| Table name | `api-data` |
| Partition key | `pk` (String) |
| Sort key | `sk` (String) |
| Billing mode | On-demand |

</td>
</tr>
</table>

> ⏳ Wait ~30 seconds for both tables to show **Active** status before proceeding.

---

## 🔐 Step 3 — IAM Role for Lambda

**Navigate:** IAM → Roles → Create role → AWS service → Lambda

Attach these 4 managed policies:

```
✅  AWSLambdaBasicExecutionRole    →  CloudWatch logging
✅  AmazonDynamoDBFullAccess       →  Read/write both tables
✅  AmazonSESFullAccess            →  Send contact emails
✅  AmazonS3FullAccess             →  Generate presigned upload URLs
```

**Role name:** `lambda-business-role`

---

## ⚡ Step 4 — Lambda Functions × 3

**Common settings for ALL 3:**

| Setting | Value |
|---------|-------|
| Runtime | Python 3.12 |
| Memory | 256 MB |
| Timeout | 30 seconds |
| Execution role | `lambda-business-role` |

**Environment variables (set on ALL 3):**

```bash
FROM_EMAIL      =  noreply@rrlearn.xyz
TO_EMAIL        =  narengl2001@gmail.com
UPLOAD_BUCKET   =  mybusiness-uploads-prod1
CONTACTS_TABLE  =  contacts
API_TABLE       =  api-data
ALLOWED_ORIGIN  =  https://rrlearn.xyz
```

---

### 📬 Lambda 1: `contact-handler`

Saves form submissions to DynamoDB and fires email via SES.

```python
import json, boto3, uuid, os
from datetime import datetime, timezone

dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
ses = boto3.client('ses', region_name='us-east-1')

FROM_EMAIL = os.environ['FROM_EMAIL']
TO_EMAIL = os.environ['TO_EMAIL']
CONTACTS_TABLE = os.environ['CONTACTS_TABLE']

def lambda_handler(event, context):
    if event.get('httpMethod') == 'OPTIONS':
        return {'statusCode': 200, 'headers': cors_headers(), 'body': ''}
    try:
        body = json.loads(event.get('body', '{}'))
        name    = body.get('name', '').strip()
        email   = body.get('email', '').strip()
        message = body.get('message', '').strip()

        if not all([name, email, message]):
            return cors_response(400, {'error': 'All fields required'})
        if len(message) > 2000:
            return cors_response(400, {'error': 'Message too long'})
        if '@' not in email or '.' not in email:
            return cors_response(400, {'error': 'Invalid email'})

        table   = dynamodb.Table(CONTACTS_TABLE)
        item_id = str(uuid.uuid4())
        table.put_item(Item={
            'id': item_id, 'name': name, 'email': email,
            'message': message,
            'submitted_at': datetime.now(timezone.utc).isoformat(),
            'status': 'new'
        })
        ses.send_email(
            Source=FROM_EMAIL,
            Destination={'ToAddresses': [TO_EMAIL]},
            Message={
                'Subject': {'Data': f'New contact from {name}'},
                'Body': {'Html': {'Data': f'From: {name}<br>{email}<br>{message}'}}
            }
        )
        return cors_response(200, {'success': True, 'message': 'Message sent!', 'id': item_id})
    except Exception as e:
        print(f'Error: {str(e)}')
        return cors_response(500, {'error': f'Error: {str(e)}'})

def cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,Authorization',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    }

def cors_response(status_code, body):
    return {'statusCode': status_code, 'headers': cors_headers(), 'body': json.dumps(body)}
```

---

### 📁 Lambda 2: `upload-handler`

Generates presigned S3 URLs — browser uploads **directly to S3**, Lambda never touches the bytes.

```python
import json, boto3, uuid, os
from datetime import datetime, timezone

s3 = boto3.client('s3', region_name='us-east-1')
UPLOAD_BUCKET = os.environ['UPLOAD_BUCKET']
ALLOWED_TYPES = {
    'image/jpeg': 'jpg', 'image/png': 'png',
    'image/webp': 'webp', 'application/pdf': 'pdf', 'text/plain': 'txt'
}
MAX_SIZE_MB = 10

def lambda_handler(event, context):
    if event.get('httpMethod') == 'OPTIONS':
        return {'statusCode': 200, 'headers': cors_headers(), 'body': ''}
    method = event.get('httpMethod', '')
    if method == 'POST': return handle_upload_request(event)
    elif method == 'GET':  return handle_get_url(event)
    return cors_response(405, {'error': 'Method not allowed'})

def handle_upload_request(event):
    try:
        body         = json.loads(event.get('body', '{}'))
        content_type = body.get('content_type', '').strip()
        file_size    = body.get('file_size', 0)

        if content_type not in ALLOWED_TYPES:
            return cors_response(400, {'error': 'File type not allowed'})
        if file_size > MAX_SIZE_MB * 1024 * 1024:
            return cors_response(400, {'error': f'Max {MAX_SIZE_MB}MB'})

        file_id  = str(uuid.uuid4())
        ext      = ALLOWED_TYPES[content_type]
        safe_key = f"uploads/{datetime.now(timezone.utc).strftime('%Y/%m/%d')}/{file_id}.{ext}"
        presigned = s3.generate_presigned_post(
            Bucket=UPLOAD_BUCKET, Key=safe_key,
            Fields={'Content-Type': content_type},
            Conditions=[
                {'Content-Type': content_type},
                ['content-length-range', 1, MAX_SIZE_MB * 1024 * 1024]
            ],
            ExpiresIn=300
        )
        return cors_response(200, {
            'upload_url':    presigned['url'],
            'upload_fields': presigned['fields'],
            'file_key':      safe_key
        })
    except Exception as e:
        return cors_response(500, {'error': str(e)})

def handle_get_url(event):
    params   = event.get('queryStringParameters') or {}
    file_key = params.get('key', '').strip()
    if not file_key.startswith('uploads/'):
        return cors_response(403, {'error': 'Access denied'})
    url = s3.generate_presigned_url(
        'get_object',
        Params={'Bucket': UPLOAD_BUCKET, 'Key': file_key},
        ExpiresIn=3600
    )
    return cors_response(200, {'download_url': url})

def cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type',
        'Access-Control-Allow-Methods': 'GET,POST,OPTIONS'
    }

def cors_response(s, b):
    return {'statusCode': s, 'headers': cors_headers(), 'body': json.dumps(b)}
```

---

### 🔄 Lambda 3: `api-handler`

Full CRUD REST API backed by DynamoDB. Decimal-safe JSON serialization included.

```python
import json, boto3, uuid, os
from datetime import datetime, timezone
from decimal import Decimal

dynamodb  = boto3.resource('dynamodb', region_name='us-east-1')
API_TABLE = os.environ['API_TABLE']

def decimal_default(obj):
    if isinstance(obj, Decimal): return float(obj)
    raise TypeError

def lambda_handler(event, context):
    if event.get('httpMethod') == 'OPTIONS':
        return {'statusCode': 200, 'headers': cors_headers(), 'body': ''}

    method = event.get('httpMethod', '')
    path   = event.get('path', '')
    params = event.get('queryStringParameters') or {}
    table  = dynamodb.Table(API_TABLE)

    try:
        if method == 'GET' and path == '/api/items':
            r = table.query(
                KeyConditionExpression='pk = :pk',
                ExpressionAttributeValues={':pk': 'ITEM'},
                ScanIndexForward=False,
                Limit=int(params.get('limit', 20))
            )
            return cors_response(200, {'items': r.get('Items', []), 'count': len(r.get('Items', []))})

        elif method == 'POST' and path == '/api/items':
            body    = json.loads(event.get('body', '{}'))
            item_id = str(uuid.uuid4())
            now     = datetime.now(timezone.utc).isoformat()
            item    = {'pk': 'ITEM', 'sk': item_id, 'id': item_id,
                       'created_at': now, 'updated_at': now, **body}
            table.put_item(Item=item)
            return cors_response(201, item)

        elif method == 'DELETE' and '/api/items/' in path:
            item_id = path.split('/')[-1]
            table.delete_item(Key={'pk': 'ITEM', 'sk': item_id})
            return cors_response(200, {'id': item_id, 'deleted': True})

        else:
            return cors_response(404, {'error': 'Route not found'})

    except Exception as e:
        return cors_response(500, {'error': str(e)})

def cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,Authorization',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    }

def cors_response(s, b):
    return {'statusCode': s, 'headers': cors_headers(), 'body': json.dumps(b, default=decimal_default)}
```

---

## 🔌 Step 5 — API Gateway

**Navigate:** API Gateway → Create API → REST API → Build

| Setting | Value |
|---------|-------|
| API name | `mybusiness-api` |
| Endpoint type | Regional |

> ✅ For **every** method: check **"Use Lambda Proxy Integration"**

| Resource | CORS | Methods | Lambda Target |
|----------|------|---------|---------------|
| `/contact` | ✅ | POST | `contact-handler` |
| `/upload` | ✅ | POST, GET | `upload-handler` |
| `/api` | ❌ | — | — |
| `/api/items` | ✅ | GET, POST | `api-handler` |
| `/api/items/{id}` | ✅ | GET, PUT, DELETE | `api-handler` |

**Deploy:** Actions → Deploy API → New Stage → `prod` → Deploy

> 📌 **Save your Invoke URL:**
> `https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod`

---

## ☁️ Step 6 — CloudFront Distribution

### Origin 1 — S3 Website

| Setting | Value |
|---------|-------|
| Origin domain | `mybusiness-website-prod1.s3.us-east-1.amazonaws.com` |
| Origin access | OAC — create new, keep defaults |
| Viewer protocol | Redirect HTTP to HTTPS |
| Cache policy | CachingOptimized |
| Root object | `index.html` |

### Origin 2 — API Gateway

| Setting | Value |
|---------|-------|
| Origin domain | `xxxxxxxxxx.execute-api.us-east-1.amazonaws.com` |
| Origin path | `/prod` |
| Protocol | HTTPS only |

### Behaviors

| Path | Origin | Methods | Cache | Request Policy |
|------|--------|---------|-------|----------------|
| `/api/*` | API GW | ALL | CachingDisabled | AllViewerExceptHostHeader |
| `/contact` | API GW | ALL | CachingDisabled | AllViewerExceptHostHeader |
| `/upload` | API GW | ALL | CachingDisabled | AllViewerExceptHostHeader |

### Error Pages

| Error | Response | Code |
|-------|----------|------|
| 403 | `/index.html` | 200 |
| 404 | `/index.html` | 200 |

> 📌 **Save your CloudFront domain:** `d1234567890.cloudfront.net`
> Wait 5–10 min for status → **Enabled**

---

## 🔒 Step 7 — S3 Bucket Policies

### 7a — Website Bucket (OAC Policy)

CloudFront → Distribution → Origins → S3 origin → Edit → **Copy policy** → Paste into S3 bucket policy

### 7b — Uploads Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowPresignedUploads",
    "Effect": "Allow",
    "Principal": "*",
    "Action": ["s3:PutObject", "s3:GetObject"],
    "Resource": "arn:aws:s3:::mybusiness-uploads-prod1/*"
  }]
}
```

### 7c — Uploads Bucket CORS

```json
[{
  "AllowedHeaders": ["*"],
  "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
  "AllowedOrigins": ["*"],
  "ExposeHeaders": ["ETag"],
  "MaxAgeSeconds": 3000
}]
```

### 7d — Invalidate Cache

```bash
aws cloudfront create-invalidation --distribution-id YOUR_DIST_ID --paths '/*'
```

---

## 🌐 Step 8 — Route 53 DNS

| Record | Name | Type | Target |
|--------|------|------|--------|
| Root domain | *(blank)* | A — Alias | CloudFront distribution |
| www | `www` | A — Alias | CloudFront distribution |

---

## 🔑 Step 9 — Custom Domain + Free SSL

**Navigate:** CloudFront → Distribution → Edit

| Setting | Value |
|---------|-------|
| Alternate domain 1 | `rrlearn.xyz` |
| Alternate domain 2 | `www.rrlearn.xyz` |
| Custom SSL cert | Your ACM certificate |

> 💡 ACM certificates are **completely free** on AWS and auto-renew forever.
> Must be issued in **us-east-1**, regardless of your app's region.

---

## 📧 Step 10 — SES Email

**Verify sending domain:**

```
1. SES → Verified identities → Create identity → Domain → rrlearn.xyz
2. SES gives 3 CNAME records for DKIM
3. Add all 3 CNAME records in Route 53
```

**Verify receiving email:**

```
1. Create identity → Email address → narengl2001@gmail.com
2. Click verification link in Gmail inbox
```

> ⚠️ SES starts in **sandbox mode** — can only send to verified addresses.
> For production: SES → Account dashboard → **Request production access**

---

## 🛡️ Step 11 — WAF Firewall

| Setting | Value |
|---------|-------|
| Name | `mybusiness-waf` |
| Resource type | CloudFront distributions |
| Region | Global (CloudFront) |

**Free managed rules to add:**

```
✅  AWSManagedRulesCommonRuleSet          →  SQLi, XSS, path traversal
✅  AWSManagedRulesKnownBadInputsRuleSet  →  Log4Shell, Spring4Shell, CVEs
✅  AWSManagedRulesAmazonIpReputationList →  Known malicious IPs
```

**Rate limit rule:**

| Setting | Value |
|---------|-------|
| Rule type | Rate-based |
| Name | `rate-limit` |
| Rate | 100 requests / 5 minutes |
| Action | Block |

---

## 📤 Step 12 — Upload Frontend

```bash
# Set your API base URL in api.js
const API_BASE = 'https://rrlearn.xyz';

# Upload to S3
aws s3 sync frontend/ s3://mybusiness-website-prod1/ --region us-east-1

# Invalidate CloudFront cache
aws cloudfront create-invalidation --distribution-id YOUR_DIST_ID --paths '/*'
```

---

## ✅ Step 13 — Test Everything

```bash
# Test website
curl -I https://rrlearn.xyz

# Test API — GET items
curl https://rrlearn.xyz/api/items

# Test API — POST new item
curl -X POST https://rrlearn.xyz/api/items \
  -H 'Content-Type: application/json' \
  -d '{"title":"Test","description":"Hello from prod"}'
```

| Test | Action | Expected |
|------|--------|----------|
| 🌐 Website loads | Open `https://rrlearn.xyz` | Full page renders |
| 📬 Contact form | Fill & submit | Success message + email received |
| 📁 File upload | Upload jpg/png/pdf | Success + file key returned |
| 🔌 API GET | `curl /api/items` | JSON items array |
| 🔌 API POST | `curl -X POST /api/items` | HTTP 201 + new item |

---

## 🐛 Troubleshooting

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| `500` on contact form | Lambda/SES error | Check CloudWatch → `/aws/lambda/contact-handler` |
| `403` on file upload | Bucket policy missing | Redo Step 7b. Block public access must be **OFF** |
| Website shows XML | OAC policy not applied | Redo Step 7a |
| API `403` from CloudFront | Missing request policy | Add `AllViewerExceptHostHeader` to each behavior |
| Domain not resolving | Route 53 wrong | Check Step 8 + verify nameservers at registrar |
| SSL error | ACM cert missing | Redo Step 9 — cert must be in `us-east-1` |
| No email received | SES sandbox | Verify `TO_EMAIL` or request production access |
| File type rejected | Not in `ALLOWED_TYPES` | Edit `upload-handler` to add more MIME types |

---

## 💰 Real Monthly Cost

| Service | Cost | Notes |
|---------|------|-------|
| CloudFront | ~$0 | Free tier: 1TB/month (12 months) |
| S3 | ~$0.02 | $0.023/GB — tiny for most sites |
| Lambda | ~$0 | Free tier: 1M requests/month |
| API Gateway | ~$0 | Free tier: 1M calls/month |
| DynamoDB | ~$0 | Free tier: 25GB storage forever |
| SES | ~$0.10 | $0.10 per 1,000 emails |
| Route 53 | $0.50 | Per hosted zone |
| ACM (SSL) | **FREE** | No charge, ever |
| **WAF** | **~$5.00** | ← Main cost, always charged |
| **Total** | **~$6–8/mo** | |

> 💡 The WAF is the only reliable cost. Everything else hits free tier for a typical portfolio or small business site.

---

## 🎯 What You've Built

A production system that:

- 🔒 **Isolates every layer** — static files private, APIs proxied, uploads presigned
- 🌍 **Delivers globally** — CloudFront edge nodes, sub-100ms worldwide
- 🛡️ **Blocks attacks** — WAF with 3 managed rule sets + rate limiting
- 📧 **Handles contact forms** — SES with DKIM verification, stored in DynamoDB
- 📁 **Manages uploads** — presigned URLs, type/size validation, dated key paths
- ⚡ **Scales to zero** — Lambda cold starts in milliseconds, pay per request
- 💸 **Costs almost nothing** — under $8/month total

---

## 👤 Author

<div align="center">

**Naren G**

*DevOps & Cloud Security Engineer*
*2+ years building production-grade cloud infrastructure*
*Open to DevOps / Cloud / SRE / Platform Engineering roles*

<br/>

[![GitHub](https://img.shields.io/badge/GitHub-Narenrohitha-181717?style=for-the-badge&logo=github)](https://github.com/Narenrohitha)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-naren--g-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/naren-g-7bb580229)
[![Medium](https://img.shields.io/badge/Medium-@narengl2001-000000?style=for-the-badge&logo=medium)](https://medium.com/@narengl2001)
[![YouTube](https://www.youtube.com/watch?v=slMSgmcaYDA&t=8s)
[![Email](https://img.shields.io/badge/Email-narengl2001@gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:narengl2001@gmail.com)

</div>

---

<div align="center">

**If this saved you hours of AWS documentation hell — leave a ⭐ and share it with your team.**

<img src="https://capsule-render.vercel.app/api?type=waving&color=FF6B00&height=100&section=footer" width="100%"/>

</div>
