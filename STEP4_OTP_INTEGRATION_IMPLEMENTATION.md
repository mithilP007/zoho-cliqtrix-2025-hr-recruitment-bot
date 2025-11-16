# Step 4: OTP Integration Implementation Guide

## Overview
This document provides the complete implementation for Step 4 of the Zoho Cliqtrix 2025 HR Recruitment Bot - OTP verification using Twilio SMS integration.

## Your Credentials
```
Zoho Recruit Access Token: 1000.1115b91e9da6f6914457c57c53e4ab31.b8adec661b8950a2e8eea5fb0419486f
Zoho Recruit Refresh Token: 1000.07612138109565076161fe0b55e13335.f884ff8ca4f7e43e57435cd4c3eb2a39
Twilio Account SID: AC351575ff88039b021e416c1b9806aa96
Twilio Auth Token: 82b113c056518b15181ae54e78bf83ec
Twilio Phone Number: +12066735201
```

## Implementation Options

Since Zoho Flow may have organization setup issues, here are THREE alternative approaches:

### Option 1: Direct Twilio API Integration (RECOMMENDED)

Integrate Twilio directly in your Zoho SalesIQ bot using Deluge scripts.

#### Step 4.1: Test Twilio SMS (Quick Verification)

First, test your Twilio credentials work:

```bash
curl -X POST "https://api.twilio.com/2010-04-01/Accounts/AC351575ff88039b021e416c1b9806aa96/Messages.json" \
  --data-urlencode "Body=Test OTP: 123456" \
  --data-urlencode "From=+12066735201" \
  --data-urlencode "To=+91YOUR_PHONE_NUMBER" \
  -u AC351575ff88039b021e416c1b9806aa96:82b113c056518b15181ae54e78bf83ec
```

Replace `+91YOUR_PHONE_NUMBER` with your actual Indian phone number.

#### Step 4.2: Create OTP Function in SalesIQ Bot

In your Zoho SalesIQ bot builder, use this Deluge script:

```deluge
// Function to generate 6-digit OTP
otpCode = randomNumber(100000, 999999);

// Store OTP with timestamp for validation
otpData = {"otp": otpCode, "timestamp": zoho.currenttime, "phone": phoneNumber};
zoho.crm.createRecord("OTP_Store", otpData);

// Send SMS via Twilio
accountSid = "AC351575ff88039b021e416c1b9806aa96";
authToken = "82b113c056518b15181ae54e78bf83ec";
fromNumber = "+12066735201";

// Build Twilio API request
twilioUrl = "https://api.twilio.com/2010-04-01/Accounts/" + accountSid + "/Messages.json";

headers = Map();
headers.put("Authorization", "Basic " + zoho.encryption.base64Encode(accountSid + ":" + authToken));

params = Map();
params.put("Body", "Your HR Bot OTP is: " + otpCode + ". Valid for 5 minutes.");
params.put("From", fromNumber);
params.put("To", phoneNumber);

// Send SMS
response = invokeurl
[
  url: twilioUrl
  type: POST
  parameters: params
  headers: headers
];

if (response.get("status") == "queued" || response.get("status") == "sent")
{
  return {"success": true, "message": "OTP sent to " + phoneNumber};
}
else
{
  return {"success": false, "message": "Failed to send OTP"};
}
```

#### Step 4.3: OTP Validation Function

```deluge
// Validate user-entered OTP
userOtp = input.get("otp");
phone = context.get("phone");

// Retrieve stored OTP
otpRecords = zoho.crm.searchRecords("OTP_Store", "(Phone:equals:" + phone + ")");

if(otpRecords.size() > 0)
{
  storedOtp = otpRecords.get(0).get("otp");
  timestamp = otpRecords.get(0).get("timestamp");
  
  // Check if OTP expired (5 minutes = 300000 milliseconds)
  currentTime = zoho.currenttime;
  timeDiff = currentTime.compareTo(timestamp);
  
  if(timeDiff > 300000)
  {
    return {"valid": false, "message": "OTP expired. Please request a new one."};
  }
  
  // Validate OTP
  if(userOtp.toString() == storedOtp.toString())
  {
    // Delete OTP after successful validation
    zoho.crm.deleteRecord("OTP_Store", otpRecords.get(0).get("id"));
    return {"valid": true, "message": "OTP verified successfully!"};
  }
  else
  {
    return {"valid": false, "message": "Invalid OTP. Please try again."};
  }
}
else
{
  return {"valid": false, "message": "No OTP found for this number."};
}
```

### Option 2: Using Zapier (FREE Alternative)

If Zoho Flow doesn't work, use Zapier's free tier:

1. Go to https://zapier.com/app/zaps
2. Create New Zap:
   - **Trigger**: Webhooks by Zapier → Catch Hook
   - Copy the webhook URL provided
   - **Action**: Twilio → Send SMS
     - Account SID: `AC351575ff88039b021e416c1b9806aa96`
     - Auth Token: `82b113c056518b15181ae54e78bf83ec`
     - From: `+12066735201`
     - To: `{{phone}}`
     - Body: `Your OTP is: {{otp}}`
3. Test and turn on the Zap
4. Use the webhook URL in your SalesIQ bot

### Option 3: Make.com (Integromat) Integration

1. Go to https://www.make.com/
2. Create new scenario:
   - **Trigger**: Webhooks → Custom Webhook
   - **Action**: Twilio → Send SMS
3. Configure with your Twilio credentials
4. Get webhook URL for bot integration

## Step 4.4: Implement in SalesIQ Bot Builder

### Complete Bot Flow with OTP

1. **Access SalesIQ**: https://salesiq.zoho.com/
2. Go to **Settings → Bots → Your Bot**
3. Add these nodes in sequence:

#### Node 1: Collect Application Data
```
Node Type: Form
Form Name: "Job Application Form"
Fields:
  - Full Name (text, required)
  - Email (email, required)
  - Phone (tel, required, placeholder: "+91")
  - Years of Experience (dropdown)
  - Resume (file upload, .pdf only)
```

#### Node 2: Send OTP
```
Node Type: Execute Code
Code: [Use the OTP generation script from Step 4.2]
Store phone number in context for validation
```

#### Node 3: Collect OTP Input
```
Node Type: Input
Question: "We've sent a 6-digit OTP to your phone. Please enter it below:"
Variable: user_otp
Validation: Must be 6 digits
```

#### Node 4: Validate OTP
```
Node Type: Execute Code
Code: [Use the OTP validation script from Step 4.3]
```

#### Node 5: Conditional Branch
```
Node Type: Condition
If OTP valid → Continue to Node 6
If OTP invalid → Return to Node 3 (max 3 attempts)
```

#### Node 6: Success Message
```
Node Type: Message
Text: "✅ Application verified! Processing your details..."
```

## Step 4.5: Testing Your Implementation

### Test Checklist:

```
☐ Test 1: Send test SMS from Twilio console
☐ Test 2: Trigger OTP from bot (use your number)
☐ Test 3: Enter correct OTP → Should pass
☐ Test 4: Enter wrong OTP → Should fail
☐ Test 5: Wait 6 minutes, try old OTP → Should expire
☐ Test 6: Request new OTP → Should send new code
☐ Test 7: Try without phone number → Should show error
```

### Testing Commands

**Test Twilio directly:**
```bash
curl -X POST "https://api.twilio.com/2010-04-01/Accounts/AC351575ff88039b021e416c1b9806aa96/Messages.json" \
  --data-urlencode "Body=Test from HR Bot: Your OTP is 123456" \
  --data-urlencode "From=+12066735201" \
  --data-urlencode "To=+91YOUR_NUMBER" \
  -u AC351575ff88039b021e416c1b9806aa96:82b113c056518b15181ae54e78bf83ec
```

## Step 4.6: Error Handling

Add error handling to your OTP functions:

```deluge
try
{
  // OTP sending code here
}
catch(e)
{
  // Log error
  zoho.crm.createRecord("Error_Log", {
    "Error_Type": "OTP_Send_Failed",
    "Error_Message": e.toString(),
    "Phone_Number": phoneNumber,
    "Timestamp": zoho.currenttime
  });
  
  return {"success": false, "message": "Unable to send OTP. Please try again or contact support."};
}
```

## Common Issues and Solutions

### Issue 1: Twilio "From number not verified"
**Solution**: In Twilio trial mode, add your test phone numbers to "Verified Caller IDs" in Twilio console.

### Issue 2: OTP not received
**Solutions**:
- Check Twilio account balance (trial gives $15 credit)
- Verify phone number format: Must be E.164 format (+919876543210)
- Check Twilio logs at https://console.twilio.com/monitor/logs/sms

### Issue 3: "Invalid Auth Token"
**Solution**: Regenerate auth token from Twilio console and update in code.

### Issue 4: Zoho Flow organization creation stuck
**Solution**: Use Option 1 (Direct Integration) or Option 2 (Zapier) instead.

## Next Steps After Step 4

Once OTP is working:

1. ✅ Step 5: Upload resume to Zoho WorkDrive
2. ✅ Step 6: Create candidate record in Zoho Recruit
3. ✅ Step 7: Link candidate to job posting
4. ✅ Step 8: Send confirmation and track application

## Resources

- **Twilio SMS Documentation**: https://www.twilio.com/docs/sms
- **Zoho SalesIQ Bot Builder**: https://www.zoho.com/salesiq/help/developer-section/bot-builder.html
- **Deluge Scripting Guide**: https://www.zoho.com/deluge/help/
- **Twilio Console**: https://console.twilio.com/
- **Test Your Bot**: https://salesiq.zoho.com/

## Support

If you encounter issues:
1. Check Twilio SMS logs for delivery status
2. Verify credentials are correct
3. Test with curl command first
4. Check phone number format (must include country code)

---

**Status**: Step 4 implementation guide complete ✅
**Created**: November 16, 2025
**Project**: Zoho Cliqtrix 2025 HR Recruitment Bot
**Developer**: Mithil P (mithilP007)
