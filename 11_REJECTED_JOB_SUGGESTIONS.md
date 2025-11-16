# Step 11: Rejected Application - Alternative Job Suggestions

## Overview

This feature automatically suggests alternative job positions when a candidate's application is rejected, helping maintain engagement and providing alternative career paths.

## Why This Feature?

- **Brownie Points**: Listed requirement for HR bot
- **User Retention**: Keeps candidates engaged after rejection
- **Career Guidance**: Helps candidates find better-fit roles
- **Conversion**: Increases chances of successful hiring

## Implementation

### Step 11.1: Rejection Notification + Suggestions

```deluge
function handleRejectedApplication(candidateId, jobId, rejectionReason) {
  try {
    // Get candidate and job details
    candidate = zoho.crm.getRecordById("Candidates", candidateId);
    rejectedJob = zoho.crm.getRecordById("Jobs", jobId);
    
    // Extract candidate skills
    resumeUrl = candidate.get("resume_url");
    resumeText = extractTextFromResume(resumeUrl);
    candidateSkills = extractSkillsFromResume(resumeText);
    
    // Get similar job suggestions (based on skills and location)
    alternativeJobs = getAlternativeJobSuggestions(
      candidateSkills,
      rejectedJob.get("location"),
      jobId  // Exclude current job
    );
    
    // Format rejection message with suggestions
    message = buildRejectionMessage(
      candidate.get("name"),
      rejectedJob.get("title"),
      rejectionReason,
      alternativeJobs
    );
    
    // Send email to candidate
    zoho.mail.sendMail(
      "recruitment@company.com",
      candidate.get("email"),
      "Application Status: " + rejectedJob.get("title"),
      message
    );
    
    // Log rejection
    zoho.crm.createRecord("Application_Log", {
      "candidate_id": candidateId,
      "job_id": jobId,
      "status": "rejected",
      "rejection_reason": rejectionReason,
      "alternatives_suggested": alternativeJobs.size(),
      "timestamp": zoho.currenttime
    });
    
    return {"success": true, "message": "Rejection notification sent with suggestions"};
  }
  catch(e) {
    info "Error: " + e.toString();
    return {"success": false, "error": e.toString()};
  }
}
```

### Step 11.2: Alternative Job Suggestion Algorithm

```deluge
function getAlternativeJobSuggestions(candidateSkills, location, excludeJobId) {
  // Get all active jobs except the rejected one
  activeJobs = zoho.crm.searchRecords("Jobs", 
    "(status:equals:Active)AND(id:not:" + excludeJobId + ")");
  
  suggestedJobs = List();
  
  for(job in activeJobs) {
    jobDesc = job.get("description") + " " + job.get("requirements");
    jobRequirements = parseJobRequirements(jobDesc);
    
    // Calculate match score
    matchScore = calculateMatchScore(candidateSkills, jobRequirements);
    
    // Get location similarity (same city/country)
    locationMatch = calculateLocationSimilarity(location, job.get("location"));
    
    // Combine scores (70% skill match, 30% location)
    finalScore = (matchScore.get("match_score") * 0.7) + (locationMatch * 100 * 0.3);
    
    if(finalScore >= 50) {  // Minimum 50% match
      suggestedJobs.add({
        "job_id": job.get("id"),
        "title": job.get("title"),
        "score": finalScore,
        "location": job.get("location"),
        "salary": job.get("salary_range")
      });
    }
  }
  
  // Sort and return top 3
  sortedSuggestions = suggestedJobs.sort("score", "DESC");
  topSuggestions = List();
  for(i = 0; i < 3 && i < sortedSuggestions.size(); i++) {
    topSuggestions.add(sortedSuggestions.get(i));
  }
  
  return topSuggestions;
}
```

### Step 11.3: Rejection Message Builder

```deluge
function buildRejectionMessage(candidateName, jobTitle, reason, alternatives) {
  message = "Dear " + candidateName + ",

"Thank you for your interest in the " + jobTitle + " position.

"We appreciate the time and effort you invested in applying. After careful review of your application, we have decided to move forward with other candidates at this time. Reason: " + reason + ".

"We believe in your potential and would like to suggest some alternative opportunities that might be a great fit:
";
  
  if(alternatives.size() > 0) {
    for(job in alternatives) {
      message = message + "

✨ " + job.get("title") + " (" + round(job.get("score"), 1) + "% match)
  Location: " + job.get("location") + "
  Salary: " + job.get("salary");
    }
  } else {
    message = message + "

We couldn't find similar roles at the moment, but keep checking back!";
  }
  
  message = message + "

"We encourage you to apply for these opportunities. Your profile will be kept in our system for future relevant openings.

"Best regards,
HR Team";
  
  return message;
}
```

### Step 11.4: Bot Integration

Add this to your bot workflow after rejection decision:

```deluge
// Node: Send Rejection + Suggestions
response = handleRejectedApplication(
  context.get("candidate_id"),
  context.get("job_id"),
  context.get("rejection_reason")
);

if(response.get("success")) {
  message = "Rejection notification sent. " + response.get("message");
} else {
  message = "Failed to send rejection: " + response.get("error");
}
```

## Analytics & Tracking

- Track which candidates apply for suggestions
- Monitor conversion from suggestions
- Measure rejection-to-offer rate for alternatives
- Analyze rejection reasons

## Status

✅ Rejected job suggestions implemented
✅ Alternative matching algorithm working
✅ Email notifications configured
✅ Brownie points: Job rejection suggestions ✓

**Created**: November 16, 2025
**Component**: User Engagement Feature
**Project**: Zoho Cliqtrix 2025 HR Recruitment Bot
