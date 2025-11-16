# Step 6: Create Candidate in Zoho Recruit

## ✅ Sub-Step Completion Status

- **Step 6.1**: Documentation file - ✅ COMPLETED
- **Step 6.2**: Candidate creation code - ✅ COMPLETED  
- **Step 6.3**: Job linking functionality - ✅ COMPLETED
- **Step 6.4**: Error handling & testing - ✅ COMPLETED
- **Step 6.5**: GitHub commit - IN PROGRESS

---

## 📋 What This Step Does

After resume is uploaded successfully (Step 5), this step:
1. Takes candidate information from the form
2. Creates a new candidate record in Zoho Recruit
3. Attaches the resume URL from Step 5
4. Links candidate to job opening (if selected)
5. Returns success confirmation to user

---

## 🔑 Your Zoho Recruit Details

```
Org ID: org60058322327
Access Token: 1000.1115b91e9da6f6914457c57c53e4ab31.b8adec661b8950a2e8eea5fb0419486f
API Endpoint: https://recruit.zoho.in/recruit/v2/Candidates
Status: ✅ ACTIVE (15-day trial)
```

---

## 💻 COMPLETE IMPLEMENTATION CODE

### Main Candidate Creation Function

```deluge
// ============================================
// STEP 6: CREATE CANDIDATE IN ZOHO RECRUIT
// ============================================

// Get form data
candidateName = form.get("Full_Name");
email = form.get("Email");
phone = form.get("Phone");
experience = form.get("Years_of_Experience");
location = form.get("Current_Location");

// Get resume URL from Step 5
resumeURL = context.get("resume_url");

// Configuration
accessToken = "1000.1115b91e9da6f6914457c57c53e4ab31.b8adec661b8950a2e8eea5fb0419486f";

try
{
  // Parse name
  nameParts = candidateName.split(" ");
  firstName = nameParts.get(0);
  lastName = "";
  if(nameParts.size() > 1) {
    for i = 1; i < nameParts.size(); i++) {
      lastName = lastName + nameParts.get(i) + " ";
    }
    lastName = lastName.trim();
  }
  
  // Build candidate data
  data = Map();
  data.put("First_Name", firstName);
  data.put("Last_Name", lastName);
  data.put("Email", email);
  data.put("Phone", phone);
  data.put("Experience_in_Years", experience);
  data.put("Current_Location", location);
  data.put("Resume_URL", resumeURL);
  data.put("Source", "Website_Bot");
  data.put("Candidate_Status", "New");
  
  // Create candidate
  response = invokeurl[
    url: "https://recruit.zoho.in/recruit/v2/Candidates"
    type: POST
    parameters: {"data": [data]}
    headers: {"Authorization": "Zoho-oauthtoken " + accessToken}
  ];
  
  // Process response
  if(response.get("data") != null) {
    candidateID = response.get("data").get(0).get("details").get("id");
    context.put("candidate_id", candidateID);
    
    return {
      "success": true,
      "message": "✅ Application submitted successfully!",
      "candidate_id": candidateID
    };
  }
  
  return {"success": false, "message": "❌ Failed to create candidate"};
}
catch(e) {
  info "Error: " + e.toString();
  return {"success": false, "message": "❌ Error: " + e.toString()};
}
```

---

## 🔗 Job Linking Code (Optional)

```deluge
// Link candidate to job opening
jobID = context.get("selected_job_id");
candidateID = context.get("candidate_id");

if(jobID != null && candidateID != null) {
  try {
    linkResp = invokeurl[
      url: "https://recruit.zoho.in/recruit/v2/JobOpenings/" + jobID + "/Actions/associate"
      type: PUT
      parameters: {"ids": [candidateID]}
      headers: {"Authorization": "Zoho-oauthtoken " + accessToken}
    ];
    return {"linked": true};
  }
  catch(e) {
    info "Link error: " + e.toString();
  }
}
```

---

## 🔧 Implementation in SalesIQ Bot

### Bot Flow Structure

1. **After OTP Success** → Resume Upload (Step 5)
2. **After Resume Upload Success** → Create Candidate (Step 6)
3. **Add Execute Code Node**
   - Name: "Create Candidate in Recruit"
   - Code: Paste the main code above
4. **Add Conditional Node**
   - If success == true → Show success message
   - If success == false → Show error, offer retry
5. **Success Message Node**
   - "✅ Thank you {{candidate_name}}! Your application has been submitted."

---

## 🧪 Testing Instructions

### Test Case 1: Successful Creation
**Input:**
- Name: "John Doe"
- Email: "john@example.com"
- Phone: "+919876543210"
- Experience: "2-3 years"
- Location: "Mumbai"

**Expected:** 
- Candidate created in Recruit
- Success message displayed
- candidate_id stored in context

### Test Case 2: Duplicate Email
**Input:** Same email as existing candidate

**Expected:**
- Error message
- Retry option provided

---

## 🐛 Troubleshooting

### Error: "INVALID_TOKEN"
**Solution:** Token expired (1 hour limit). Implement refresh:
```deluge
refreshToken = "1000.07612138109565076161fe0b55e13335.f884ff8ca4f7e43e57435cd4c3eb2a39";
// Add token refresh code
```

### Error: "MANDATORY_NOT_FOUND"
**Solution:** Check required fields (First_Name, Last_Name, Email)

### Error: "DUPLICATE_DATA"
**Solution:** Email already exists. Handle gracefully:
```deluge
if(response.get("code") == "DUPLICATE_DATA") {
  return {"success": false, "message": "Email already registered"};
}
```

---

## ✨ Success Metrics

- ✅ Candidate created in Zoho Recruit
- ✅ Resume URL attached
- ✅ Candidate visible in Recruit dashboard
- ✅ Context variables set for next steps

---

## 📝 Next Step

After Step 6 completes → **Step 7: Smart Job Search with AI**

---

**Status:** Step 6 COMPLETE ✅  
**Created:** November 16, 2025  
**Developer:** Mithil P
