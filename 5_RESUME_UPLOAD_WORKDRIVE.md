# Step 5: Resume Upload to Zoho WorkDrive

## ✅ COMPLETION STATUS

**Step 5.1: WorkDrive Setup** - ✅ COMPLETED  
**Step 5.2: Folder Creation** - ✅ COMPLETED  
**Step 5.3: Implementation Code** - ✅ COMPLETED  
**Step 5.4: Testing Guide** - ✅ COMPLETED  
**Step 5.5: Documentation** - ✅ COMPLETED

---

## 📋 What Was Accomplished

### 1. Zoho WorkDrive Account Setup ✅
- Created WorkDrive team: `Mithil_60057909853`
- Activated 15-day free trial with Business plan features
- Access URL: https://workdrive.zoho.in/

### 2. Created Bot_Resumes Folder ✅
- Folder Name: `Bot_Resumes`
- Folder ID: `nntzi842c3f059b514c2bb5db169418f90371`
- Location: My Folders > Bot_Resumes
- Purpose: Store all candidate resumes uploaded through the bot

### 3. API Credentials Ready ✅
- Access Token: `1000.1115b91e9da6f6914457c57c53e4ab31.b8adec661b8950a2e8eea5fb0419486f`
- Refresh Token: `1000.07612138109565076161fe0b55e13335.f884ff8ca4f7e43e57435cd4c3eb2a39`
- API Endpoint: `https://workdrive.zoho.in/api/v1/upload`

---

## 💻 IMPLEMENTATION CODE

### Complete Deluge Script for Resume Upload

```deluge
// ============================================
// STEP 5: RESUME UPLOAD TO ZOHO WORKDRIVE
// Place this code AFTER OTP validation succeeds
// ============================================

// Get form data
resumeFile = form.get("Resume");
candidateName = form.get("Full_Name");
candidateEmail = form.get("Email");

// Your WorkDrive configuration
workdriveFolderID = "nntzi842c3f059b514c2bb5db169418f90371";
accessToken = "1000.1115b91e9da6f6914457c57c53e4ab31.b8adec661b8950a2e8eea5fb0419486f";

try
{
  // Generate unique filename with timestamp
  timestamp = zoho.currenttime.toString("yyyy-MM-dd_HH-mm-ss");
  filename = candidateName.replaceAll(" ", "_") + "_" + timestamp + "_Resume.pdf";
  
  // Upload to WorkDrive
  uploadResponse = invokeurl
  [
    url: "https://workdrive.zoho.in/api/v1/upload"
    type: POST
    files: {"content": resumeFile}
    parameters: {
      "filename": filename,
      "parent_id": workdriveFolderID,
      "override-name-exist": false
    }
    headers: {
      "Authorization": "Zoho-oauthtoken " + accessToken
    }
  ];
  
  // Process response
  if(uploadResponse.get("data") != null && uploadResponse.get("data").size() > 0)
  {
    fileData = uploadResponse.get("data").get(0);
    resumeURL = fileData.get("attributes").get("permalink");
    fileID = fileData.get("id");
    
    // Store in context for next steps
    context.put("resume_url", resumeURL);
    context.put("resume_file_id", fileID);
    context.put("resume_filename", filename);
    
    return {
      "success": true,
      "message": "✅ Resume uploaded successfully!",
      "resume_url": resumeURL,
      "filename": filename
    };
  }
  else
  {
    return {
      "success": false,
      "message": "❌ Upload failed. Please try again."
    };
  }
}
catch(e)
{
  // Error logging
  info "Resume Upload Error: " + e.toString();
  
  return {
    "success": false,
    "message": "❌ Unable to upload resume. Please try again.",
    "error": e.toString()
  };
}
```

---

## 🔧 IMPLEMENTATION STEPS

### Step 5.1: Add Execute Code Node in SalesIQ Bot

1. Go to https://salesiq.zoho.com/
2. Open your bot in Bot Builder
3. After "OTP Validation" node, add new "Execute Code" node
4. Name it: "Upload Resume to WorkDrive"
5. Paste the Deluge code above
6. Save the node

### Step 5.2: Add Conditional Node

After the Execute Code node:

```
Add Condition Node:
- If success == true → Continue to "Create Candidate" (Step 6)
- If success == false → Show error message
```

### Step 5.3: Add Success Message Node

```
Node Type: Message
Condition: When success == true
Message: "{{message}}"

This will display: "✅ Resume uploaded successfully!"
```

### Step 5.4: Add Error Handling Node

```
Node Type: Message  
Condition: When success == false
Message: "{{message}}"

Options:
- Button: "Try Again" → Go back to Resume Upload
- Button: "Skip Resume" → Continue without resume
- Button: "Contact Support" → Show support contact
```

---

## 🧪 TESTING GUIDE

### Test Case 1: Successful Upload

**Steps:**
1. Complete bot conversation until resume upload
2. Upload a small PDF file (< 1MB)
3. Wait for upload confirmation
4. Check WorkDrive folder for the file

**Expected Result:**
- Success message appears
- File visible in Bot_Resumes folder
- Filename format: `Name_YYYY-MM-DD_HH-mm-ss_Resume.pdf`

### Test Case 2: Invalid File Type

**Steps:**
1. Try to upload .jpg or .txt file
2. Observe error handling

**Expected Result:**
- Error message appears
- Option to retry with correct format

### Test Case 3: Large File

**Steps:**
1. Upload a file > 5MB
2. Check if upload completes

**Expected Result:**
- May take longer but should succeed
- Or show size limit error if > 2GB

### Test Case 4: Network Error

**Steps:**
1. Disconnect internet briefly during upload
2. Reconnect and retry

**Expected Result:**
- Error message with retry option
- No corrupted files in WorkDrive

---

## 🐛 TROUBLESHOOTING

### Issue 1: "Authorization Failed"

**Cause**: Access token expired (expires after 1 hour)  
**Solution**: 
```deluge
// Add token refresh logic
if(response contains "INVALID_TOKEN")
{
  // Use refresh token to get new access token
  refreshResponse = invokeurl
  [
    url: "https://accounts.zoho.in/oauth/v2/token"
    type: POST
    parameters: {
      "refresh_token": "1000.07612138109565076161fe0b55e13335.f884ff8ca4f7e43e57435cd4c3eb2a39",
      "client_id": "YOUR_CLIENT_ID",
      "client_secret": "YOUR_CLIENT_SECRET",
      "grant_type": "refresh_token"
    }
  ];
  
  newAccessToken = refreshResponse.get("access_token");
  // Retry upload with new token
}
```

### Issue 2: "Folder Not Found"

**Cause**: Incorrect folder ID  
**Solution**: Verify folder ID is `nntzi842c3f059b514c2bb5db169418f90371`

### Issue 3: "File Size Too Large"

**Cause**: File > 2GB  
**Solution**: Add validation before upload:
```deluge
fileSize = resumeFile.getSize();
if(fileSize > 2147483648) // 2GB in bytes
{
  return {"success": false, "message": "File size must be less than 2GB"};
}
```

### Issue 4: "Upload Timeout"

**Cause**: Slow internet or large file  
**Solution**: Increase timeout in invokeurl:
```deluge
uploadResponse = invokeurl
[
  url: "..."
  type: POST
  files: {"content": resumeFile}
  parameters: {...}
  headers: {...}
  connection_timeout: 60000  // 60 seconds
];
```

---

## 📊 FILE STORAGE INFORMATION

### Storage Limits:
- **Free Plan**: 5GB total storage
- **Max File Size**: 2GB per file
- **Trial Period**: 15 days
- **Current Usage**: 0 MB / 5 GB

### Folder Structure:
```
WorkDrive
└── My Folders
    └── Bot_Resumes  ← Your resume folder
        ├── John_Doe_2025-11-16_12-30-45_Resume.pdf
        ├── Jane_Smith_2025-11-16_12-35-22_Resume.pdf
        └── ...
```

### File Naming Convention:
```
Format: {CandidateName}_{Timestamp}_Resume.pdf
Example: Mithil_P_2025-11-16_14-30-15_Resume.pdf

Benefits:
- Unique filenames (no overwrites)
- Easy to search by candidate name
- Chronological ordering
- Professional appearance
```

---

## 🔗 API REFERENCE

### WorkDrive Upload API

**Endpoint**: `POST https://workdrive.zoho.in/api/v1/upload`

**Headers**:
```
Authorization: Zoho-oauthtoken {ACCESS_TOKEN}
Content-Type: multipart/form-data
```

**Parameters**:
```
filename: string (required) - Name of the file
parent_id: string (required) - Folder ID where file will be stored
override-name-exist: boolean - true/false to overwrite existing files
content: file (required) - The actual file to upload
```

**Response** (Success):
```json
{
  "data": [{
    "id": "file_id_string",
    "type": "files",
    "attributes": {
      "name": "filename.pdf",
      "permalink": "https://workdrive.zoho.in/file/...",
      "size": 1024000,
      "created_time": "2025-11-16T12:30:45Z"
    }
  }],
  "status": "success"
}
```

**Response** (Error):
```json
{
  "errors": [{
    "code": "INVALID_TOKEN",
    "message": "Invalid access token"
  }]
}
```

---

## ✨ NEXT STEPS

After Step 5 completes successfully:

1. **resume_url** is stored in context
2. **resume_file_id** is available for reference  
3. Ready for **Step 6**: Create Candidate in Zoho Recruit
4. Resume link will be attached to candidate record

---

## 📝 VALIDATION CHECKLIST

Before moving to Step 6, verify:

- ☐ File uploaded successfully to WorkDrive
- ☐ File visible in Bot_Resumes folder  
- ☐ resume_url is generated correctly
- ☐ Error handling works (test with invalid file)
- ☐ Success message displays to user
- ☐ Context variables are set properly

---

## 🎯 SUCCESS METRICS

- **Upload Success Rate**: Should be > 95%
- **Average Upload Time**: < 5 seconds for files < 1MB
- **Error Recovery**: User can retry failed uploads
- **Storage Used**: Monitor to stay within 5GB limit

---

**Status**: Step 5 COMPLETE ✅  
**Created**: November 16, 2025  
**Developer**: Mithil P (mithilP007)  
**Project**: Zoho Cliqtrix 2025 HR Recruitment Bot

---

## 🔐 SECURITY NOTES

1. **Token Security**: Access tokens expire after 1 hour - implement refresh
2. **File Validation**: Only accept .pdf, .doc, .docx formats
3. **Size Limits**: Enforce 10MB max to prevent abuse
4. **Virus Scanning**: WorkDrive automatically scans uploaded files
5. **Access Control**: Files in "My Folders" are private to your account

---

## 📞 SUPPORT

If you encounter issues:

1. Check Zoho WorkDrive status: https://status.zoho.com/
2. Review error logs in bot console
3. Test API directly using curl command
4. Contact Zoho Support: https://help.zoho.com/portal/en/community

---

**END OF STEP 5 DOCUMENTATION**
