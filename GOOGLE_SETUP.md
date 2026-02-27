# Google Services Setup Guide

Complete guide for connecting Gmail, Google Calendar, and Google Maps APIs to your Daily Digest application.

## Overview

Google services require OAuth 2.0 authentication. You'll need to:
1. Create a Google Cloud project
2. Enable APIs
3. Create OAuth credentials
4. Generate access/refresh tokens
5. Configure your application

---

## Part 1: Google Cloud Console Setup

### Step 1: Create a Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click **"Select a project"** → **"New Project"**
3. Name it: `daily-digest-api`
4. Click **"Create"**

### Step 2: Enable Required APIs

In your project dashboard:

1. **Gmail API**:
   - Navigate to **"APIs & Services"** → **"Library"**
   - Search for "Gmail API"
   - Click **"Enable"**

2. **Google Calendar API**:
   - Search for "Google Calendar API"
   - Click **"Enable"**

3. **Google Maps APIs** (for traffic):
   - Search for "Routes API"
   - Click **"Enable"**
   - This is the newer Routes API (v2) for traffic and directions

---

## Part 2: OAuth 2.0 Credentials

### Step 1: Configure OAuth Consent Screen

1. Go to **"APIs & Services"** → **"OAuth consent screen"**
2. Choose **"External"** (unless you have Google Workspace)
3. Fill in required fields:
   - **App name**: Daily Digest API
   - **User support email**: Your email
   - **Developer contact**: Your email
4. Click **"Save and Continue"**

### Step 2: Add Scopes

1. Click **"Add or Remove Scopes"**
2. Add these scopes:
   ```
   https://www.googleapis.com/auth/gmail.readonly
   https://www.googleapis.com/auth/calendar.readonly
   https://www.googleapis.com/auth/userinfo.email
   ```
3. Click **"Update"** → **"Save and Continue"**

### Step 3: Add Test Users (for development)

1. Click **"Add Users"**
2. Add your Gmail address
3. Click **"Save and Continue"**

### Step 4: Create OAuth Client ID

1. Go to **"APIs & Services"** → **"Credentials"**
2. Click **"Create Credentials"** → **"OAuth client ID"**
3. Choose **"Desktop app"** (or "Web application" for production)
4. Name it: `daily-digest-desktop-client`
5. Click **"Create"**
6. **Download the JSON file** or copy:
   - Client ID
   - Client Secret

---

## Part 3: Generate Refresh Token

You need a refresh token to access Google APIs without user interaction.

### Option A: Using Python Script (Recommended)

Create `setup/get_google_tokens.py`:

```python
#!/usr/bin/env python3
"""
Script to generate Google OAuth refresh token
Run once to get your refresh token for Gmail and Calendar
"""

from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
import json

# Scopes for Gmail and Calendar read access
SCOPES = [
    'https://www.googleapis.com/auth/gmail.readonly',
    'https://www.googleapis.com/auth/calendar.readonly',
]

def get_refresh_token():
    """Generate and display refresh token"""
    
    # You'll need to create this credentials.json from Google Cloud Console
    flow = InstalledAppFlow.from_client_secrets_file(
        'credentials.json',  # Download this from Google Cloud Console
        SCOPES
    )
    
    # Run local server for OAuth flow
    creds = flow.run_local_server(port=0)
    
    print("\n" + "="*50)
    print("SUCCESS! Here are your credentials:")
    print("="*50)
    print(f"\nRefresh Token: {creds.refresh_token}")
    print(f"Client ID: {creds.client_id}")
    print(f"Client Secret: {creds.client_secret}")
    print("\nAdd these to your .env file:")
    print("="*50)
    print(f"GMAIL_CLIENT_ID={creds.client_id}")
    print(f"GMAIL_CLIENT_SECRET={creds.client_secret}")
    print(f"GMAIL_REFRESH_TOKEN={creds.refresh_token}")
    print("="*50)
    
    # Save for future use
    with open('token.json', 'w') as token_file:
        token_data = {
            'refresh_token': creds.refresh_token,
            'client_id': creds.client_id,
            'client_secret': creds.client_secret,
        }
        json.dump(token_data, token_file, indent=2)
    
    print("\nCredentials also saved to token.json")
    print("IMPORTANT: Keep this file secure and never commit it!")

if __name__ == '__main__':
    print("Google OAuth Token Generator")
    print("="*50)
    print("\nThis will open a browser window for authentication.")
    print("Make sure you have credentials.json from Google Cloud Console.\n")
    
    try:
        get_refresh_token()
    except FileNotFoundError:
        print("\n❌ ERROR: credentials.json not found!")
        print("\nSteps to get credentials.json:")
        print("1. Go to Google Cloud Console")
        print("2. APIs & Services → Credentials")
        print("3. Create OAuth 2.0 Client ID (Desktop app)")
        print("4. Download JSON and save as 'credentials.json'")
    except Exception as e:
        print(f"\n❌ ERROR: {str(e)}")
```

### Run the Token Generator:

```bash
# Install required package
pip install google-auth-oauthlib

# Download credentials.json from Google Cloud Console
# Place it in your project root

# Run the script
python3 setup/get_google_tokens.py
```

This will:
1. Open your browser
2. Ask you to sign in with Google
3. Request permissions
4. Generate and display your refresh token

### Option B: Using OAuth Playground

1. Go to [OAuth 2.0 Playground](https://developers.google.com/oauthplayground/)
2. Click the gear icon (⚙️) → **"Use your own OAuth credentials"**
3. Enter your Client ID and Client Secret
4. In Step 1, select:
   - `Gmail API v1` → `https://www.googleapis.com/auth/gmail.readonly`
   - `Calendar API v3` → `https://www.googleapis.com/auth/calendar.readonly`
5. Click **"Authorize APIs"**
6. Sign in with your Google account
7. In Step 2, click **"Exchange authorization code for tokens"**
8. Copy the **refresh_token**

---

## Part 4: Google Routes API Key

For traffic data, you need an API key (simpler than OAuth).

1. Go to **"APIs & Services"** → **"Credentials"**
2. Click **"Create Credentials"** → **"API key"**
3. Copy the API key
4. (Optional) Click **"Restrict Key"**:
   - API restrictions → Select "Routes API"
   - Application restrictions → Set IP restrictions (your server IP)

**Note**: Routes API is the newer v2 API for routing and traffic. It provides better real-time traffic data than the older Distance Matrix API.

See **[ROUTES_API_SETUP.md](ROUTES_API_SETUP.md)** for detailed configuration and route setup.

---

## Part 5: Update Your Application

### Update `.env` file:

```bash
# Gmail & Calendar (OAuth)
GMAIL_CLIENT_ID=your_client_id_here
GMAIL_CLIENT_SECRET=your_client_secret_here
GMAIL_REFRESH_TOKEN=your_refresh_token_here

# Google Calendar
GOOGLE_CALENDAR_CREDENTIALS=your_refresh_token_here
# Note: You can use the same refresh token for both

# Google Maps (API Key)
GOOGLE_API_KEY=your_google_maps_api_key_here
TRAFFIC_API_KEY=your_google_maps_api_key_here
```

### Update Email Service

Update `services/email_service.py` to use actual Gmail API:

```python
from google.oauth2.credentials import Credentials
from google.auth.transport.requests import Request
from googleapiclient.discovery import build
import base64

async def fetch_recent_emails(self, max_results: int = None):
    """Fetch recent unread emails from Gmail"""
    try:
        creds = Credentials(
            token=None,
            refresh_token=self.refresh_token,
            client_id=self.client_id,
            client_secret=self.client_secret,
            token_uri='https://oauth2.googleapis.com/token'
        )
        
        # Refresh token if needed
        if creds.expired or not creds.valid:
            creds.refresh(Request())
        
        service = build('gmail', 'v1', credentials=creds)
        
        # Get list of messages
        results = service.users().messages().list(
            userId='me',
            labelIds=['INBOX', 'UNREAD'],
            maxResults=max_results or settings.MAX_EMAILS_TO_FETCH
        ).execute()
        
        messages = results.get('messages', [])
        emails = []
        
        # Fetch each message details
        for msg in messages:
            message = service.users().messages().get(
                userId='me',
                id=msg['id'],
                format='metadata',
                metadataHeaders=['From', 'Subject', 'Date']
            ).execute()
            
            headers = message['payload']['headers']
            from_email = next(h['value'] for h in headers if h['name'] == 'From')
            subject = next(h['value'] for h in headers if h['name'] == 'Subject')
            date = next(h['value'] for h in headers if h['name'] == 'Date')
            
            # Get snippet
            snippet = message.get('snippet', '')
            
            emails.append({
                'from': from_email,
                'subject': subject,
                'snippet': snippet,
                'date': date,
                'id': msg['id']
            })
        
        return emails
        
    except Exception as e:
        logger.error(f"Error fetching emails: {str(e)}")
        return self._get_sample_emails()
```

### Update Calendar Service

Update `services/calendar_service.py`:

```python
from google.oauth2.credentials import Credentials
from google.auth.transport.requests import Request
from googleapiclient.discovery import build
from datetime import datetime, timedelta

async def fetch_today_events(self):
    """Fetch today's calendar events"""
    try:
        creds = Credentials(
            token=None,
            refresh_token=self.credentials,
            client_id=settings.GMAIL_CLIENT_ID,
            client_secret=settings.GMAIL_CLIENT_SECRET,
            token_uri='https://oauth2.googleapis.com/token'
        )
        
        if creds.expired or not creds.valid:
            creds.refresh(Request())
        
        service = build('calendar', 'v3', credentials=creds)
        
        # Get today's date range
        now = datetime.utcnow().isoformat() + 'Z'
        end_of_day = (datetime.utcnow() + timedelta(days=1)).replace(
            hour=0, minute=0, second=0, microsecond=0
        ).isoformat() + 'Z'
        
        # Fetch events
        events_result = service.events().list(
            calendarId='primary',
            timeMin=now,
            timeMax=end_of_day,
            singleEvents=True,
            orderBy='startTime'
        ).execute()
        
        events = events_result.get('items', [])
        
        return [
            {
                'time': event['start'].get('dateTime', event['start'].get('date')),
                'title': event.get('summary', 'No title'),
                'description': event.get('description', ''),
                'location': event.get('location', ''),
                'duration': 'TBD'  # Calculate from start/end
            }
            for event in events
        ]
        
    except Exception as e:
        logger.error(f"Error fetching calendar: {str(e)}")
        return self._get_sample_events()
```

---

## Part 6: Testing

### Test Gmail Connection:

```python
# test_gmail.py
from services.email_service import EmailService
import asyncio

async def test():
    service = EmailService()
    if service.is_configured():
        emails = await service.fetch_recent_emails(max_results=5)
        print(f"Found {len(emails)} emails")
        for email in emails:
            print(f"  - {email['subject']}")
    else:
        print("Gmail not configured")

asyncio.run(test())
```

### Test Calendar Connection:

```python
# test_calendar.py
from services.calendar_service import CalendarService
import asyncio

async def test():
    service = CalendarService()
    if service.is_configured():
        events = await service.fetch_today_events()
        print(f"Found {len(events)} events today")
        for event in events:
            print(f"  - {event['title']} at {event['time']}")
    else:
        print("Calendar not configured")

asyncio.run(test())
```

---

## Security Best Practices

1. **Never commit credentials**:
   ```bash
   # Add to .gitignore
   .env
   credentials.json
   token.json
   *credentials*.json
   ```

2. **Use environment variables** in production

3. **Rotate tokens regularly**:
   - Revoke old tokens in Google Cloud Console
   - Generate new ones

4. **Restrict API keys**:
   - Limit to specific APIs
   - Set IP restrictions
   - Use HTTP referrer restrictions for web apps

5. **Monitor usage**:
   - Check Google Cloud Console quotas
   - Set up billing alerts
   - Monitor for unusual activity

---

## Troubleshooting

### "invalid_grant" error:
- Refresh token expired or revoked
- Re-run the token generator script
- Check if you removed user from test users

### "Access denied" errors:
- Make sure APIs are enabled in Google Cloud
- Check OAuth scopes match what you requested
- Verify user is added to test users (for external apps)

### "Quota exceeded" errors:
- Check quotas in Google Cloud Console
- Request quota increases if needed
- Implement rate limiting in your app

### Token refresh failures:
- Verify client ID and secret are correct
- Check if OAuth consent screen is configured
- Ensure refresh token hasn't been revoked

---

## Cost Considerations

### Free Tier Limits (as of 2026):

- **Gmail API**: 1 billion quota units/day (free)
- **Calendar API**: 1 million requests/day (free)
- **Google Maps APIs**: 
  - Distance Matrix: $5-10 per 1000 requests
  - First $200/month free credit

### Optimization Tips:

1. **Cache responses** using Redis
2. **Batch requests** when possible
3. **Use partial responses** to reduce data transfer
4. **Implement exponential backoff** for retries
5. **Monitor usage** in Google Cloud Console

---

## Next Steps

1. ✅ Set up Google Cloud project
2. ✅ Enable APIs
3. ✅ Create OAuth credentials
4. ✅ Generate refresh token
5. ✅ Update `.env` file
6. ✅ Test connections
7. 🚀 Deploy to production

For other services (NewsAPI, Weather, Todoist), check their respective documentation - they typically just need an API key!
