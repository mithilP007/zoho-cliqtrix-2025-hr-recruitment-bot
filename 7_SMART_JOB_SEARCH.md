# STEP 7: Smart Job Search with Skills Matching

## Overview
This step implements an intelligent job search and matching system within the Zoho Cliqtrix 2025 HR Recruitment Bot. The system analyzes candidate skills and experience to provide personalized job recommendations.

## Architecture

### Components
1. **Job Search Module**: Queries available job openings from Zoho Recruit
2. **Skills Matching Engine**: Compares candidate skills with job requirements
3. **Ranking Algorithm**: Ranks jobs by relevance and match percentage
4. **AI-Powered Matching**: Uses Zia AI for intelligent skill extraction and analysis
5. **Result Display**: Formats results for bot carousel UI

## Implementation

### Configuration Variables
```javascript
// Zoho Recruit API Configuration
String accessToken = "1000.1115b91e9da6f6914457c57c53e4ab31.b8adec661b8950a2e8eea5fb0419486f";
String refreshToken = "1000.07612138109565076161fe0b55e13335.f884ff8ca4f7e43e57435cd4c3eb2a39";
String orgId = "org60058322327";
String apiBaseUrl = "https://recruit.zoho.in/api/v2";

// Job Search Parameters
String candidateId = ""; // From Step 6
Integer maxResults = 10;
Double minMatchPercentage = 60.0; // Minimum 60% match
```

### Function 1: Query Available Jobs
```javascript
function queryAvailableJobs() {
  try {
    // API endpoint to fetch jobs
    String endpoint = apiBaseUrl + "/jobs?organization_id=" + orgId + "&status=Open&per_page=50";
    
    Map<String, String> headers = Map();
    headers.put("Authorization", "Zoho-oauthtoken " + accessToken);
    headers.put("Content-Type", "application/json");
    
    // Make API call
    Map<String, Object> response = invokeUrl(
      [
        url: endpoint,
        type: "GET",
        headers: headers,
        connection: "zoho_recruit_connection"
      ]
    );
    
    if (response.get("code") == 200) {
      List<Map<String, Object>> jobs = response.get("data");
      info("Found " + jobs.size() + " open positions");
      return jobs;
    } else {
      error("Failed to query jobs: " + response.get("message"));
      return null;
    }
  } catch (Exception e) {
    error("Exception in queryAvailableJobs: " + e.getMessage());
    return null;
  }
}
```

### Function 2: Extract Candidate Skills
```javascript
function extractCandidateSkills(String candidateId) {
  try {
    // Fetch candidate details
    String endpoint = apiBaseUrl + "/candidates/" + candidateId + "?organization_id=" + orgId;
    
    Map<String, String> headers = Map();
    headers.put("Authorization", "Zoho-oauthtoken " + accessToken);
    headers.put("Content-Type", "application/json");
    
    Map<String, Object> candidateResponse = invokeUrl(
      [
        url: endpoint,
        type: "GET",
        headers: headers,
        connection: "zoho_recruit_connection"
      ]
    );
    
    if (candidateResponse.get("code") == 200) {
      Map<String, Object> candidateData = candidateResponse.get("data");
      
      // Extract skills from candidate profile
      List<String> skills = candidateData.get("skills");
      String experience = candidateData.get("experience");
      String location = candidateData.get("location");
      List<String> qualifications = candidateData.get("qualifications");
      
      Map<String, Object> candidateProfile = Map();
      candidateProfile.put("skills", skills);
      candidateProfile.put("experience", experience);
      candidateProfile.put("location", location);
      candidateProfile.put("qualifications", qualifications);
      
      info("Extracted candidate profile successfully");
      return candidateProfile;
    } else {
      error("Failed to fetch candidate: " + candidateResponse.get("message"));
      return null;
    }
  } catch (Exception e) {
    error("Exception in extractCandidateSkills: " + e.getMessage());
    return null;
  }
}
```

### Function 3: Calculate Skill Match Percentage
```javascript
function calculateSkillMatch(Map<String, Object> candidateProfile, Map<String, Object> job) {
  try {
    List<String> candidateSkills = candidateProfile.get("skills");
    List<String> jobRequiredSkills = job.get("required_skills");
    
    if (candidateSkills == null || jobRequiredSkills == null) {
      return 0.0;
    }
    
    Integer matchCount = 0;
    for (String skill : jobRequiredSkills) {
      if (candidateSkills.contains(skill)) {
        matchCount++;
      }
    }
    
    Double matchPercentage = (matchCount / jobRequiredSkills.size()) * 100;
    
    // Location bonus
    String candidateLocation = candidateProfile.get("location");
    String jobLocation = job.get("location");
    
    if (candidateLocation.toLowerCase().equalsIgnoreCase(jobLocation.toLowerCase())) {
      matchPercentage = matchPercentage * 1.1; // 10% bonus for location match
    }
    
    // Cap at 100%
    if (matchPercentage > 100.0) {
      matchPercentage = 100.0;
    }
    
    return matchPercentage;
  } catch (Exception e) {
    error("Exception in calculateSkillMatch: " + e.getMessage());
    return 0.0;
  }
}
```

### Function 4: Rank Jobs by Relevance
```javascript
function rankJobsByRelevance(List<Map<String, Object>> jobs, Map<String, Object> candidateProfile) {
  try {
    List<Map<String, Object>> rankedJobs = List();
    
    for (Map<String, Object> job : jobs) {
      Double matchPercentage = calculateSkillMatch(candidateProfile, job);
      
      if (matchPercentage >= minMatchPercentage) {
        job.put("matchPercentage", matchPercentage);
        rankedJobs.add(job);
      }
    }
    
    // Sort by match percentage (highest first)
    rankedJobs.sort(
      comparator(
        a, b -> {
          return (b.get("matchPercentage") <> a.get("matchPercentage"));
        }
      )
    );
    
    // Limit to top results
    if (rankedJobs.size() > maxResults) {
      rankedJobs = rankedJobs.subList(0, maxResults);
    }
    
    info("Ranked " + rankedJobs.size() + " relevant jobs");
    return rankedJobs;
  } catch (Exception e) {
    error("Exception in rankJobsByRelevance: " + e.getMessage());
    return null;
  }
}
```

### Function 5: Format Results for Carousel
```javascript
function formatJobsForCarousel(List<Map<String, Object>> rankedJobs) {
  try {
    List<Map<String, Object>> carouselItems = List();
    
    for (Map<String, Object> job : rankedJobs) {
      Map<String, Object> carouselItem = Map();
      
      carouselItem.put("title", job.get("job_title"));
      carouselItem.put("subtitle", job.get("company_name") + " | " + job.get("location"));
      carouselItem.put("description", job.get("job_description"));
      
      // Add match percentage badge
      Double matchPercentage = job.get("matchPercentage");
      String matchBadge = "Match: " + matchPercentage.round(2) + "%";
      carouselItem.put("matchBadge", matchBadge);
      
      // Add action buttons
      List<Map<String, String>> buttons = List();
      
      Map<String, String> applyButton = Map();
      applyButton.put("label", "Apply Now");
      applyButton.put("action", "apply_job");
      applyButton.put("job_id", job.get("id"));
      buttons.add(applyButton);
      
      Map<String, String> detailsButton = Map();
      detailsButton.put("label", "View Details");
      detailsButton.put("action", "view_details");
      detailsButton.put("job_id", job.get("id"));
      buttons.add(detailsButton);
      
      Map<String, String> skipButton = Map();
      skipButton.put("label", "Skip");
      skipButton.put("action", "skip_job");
      buttons.add(skipButton);
      
      carouselItem.put("buttons", buttons);
      carouselItems.add(carouselItem);
    }
    
    info("Formatted " + carouselItems.size() + " jobs for carousel display");
    return carouselItems;
  } catch (Exception e) {
    error("Exception in formatJobsForCarousel: " + e.getMessage());
    return null;
  }
}
```

### Function 6: Main Job Search Function
```javascript
function performSmartJobSearch(String candidateId) {
  try {
    info("Starting smart job search for candidate: " + candidateId);
    
    // Step 1: Get candidate profile
    Map<String, Object> candidateProfile = extractCandidateSkills(candidateId);
    if (candidateProfile == null) {
      return Map();
    }
    
    // Step 2: Query available jobs
    List<Map<String, Object>> allJobs = queryAvailableJobs();
    if (allJobs == null || allJobs.isEmpty()) {
      info("No jobs available");
      return Map();
    }
    
    // Step 3: Rank jobs
    List<Map<String, Object>> rankedJobs = rankJobsByRelevance(allJobs, candidateProfile);
    if (rankedJobs == null || rankedJobs.isEmpty()) {
      info("No matching jobs found");
      return Map();
    }
    
    // Step 4: Format for display
    List<Map<String, Object>> carouselItems = formatJobsForCarousel(rankedJobs);
    
    // Step 5: Return results
    Map<String, Object> result = Map();
    result.put("success", true);
    result.put("jobsFound", rankedJobs.size());
    result.put("carouselItems", carouselItems);
    result.put("candidateProfile", candidateProfile);
    
    return result;
  } catch (Exception e) {
    error("Exception in performSmartJobSearch: " + e.getMessage());
    return Map();
  }
}
```

## Testing

### Test Case 1: Query Available Jobs
**Input**: Organization ID and access token
**Expected Output**: List of open job positions
**Validation**: 
- Response code is 200
- Jobs list is not empty
- Each job has required fields (job_title, location, required_skills)

### Test Case 2: Extract Candidate Skills
**Input**: Candidate ID from Step 6
**Expected Output**: Candidate profile with skills, experience, location
**Validation**:
- Skills array is populated
- Location is valid
- Qualifications are retrieved

### Test Case 3: Calculate Skill Match
**Input**: Candidate profile and job
**Expected Output**: Match percentage (0-100%)
**Validation**:
- Match percentage is between 0 and 100
- Location match gives bonus
- Skills are correctly compared

### Test Case 4: Rank Jobs
**Input**: All available jobs and candidate profile
**Expected Output**: Top 10 jobs ranked by relevance
**Validation**:
- Jobs are sorted by match percentage
- Jobs below 60% match are filtered out
- Results limited to maxResults (10)

### Test Case 5: Format Results
**Input**: Ranked jobs list
**Expected Output**: Carousel-formatted items with buttons
**Validation**:
- All required fields are present
- Buttons have correct action names
- Match percentage badge is calculated

### Test Case 6: End-to-End Smart Job Search
**Input**: Candidate ID
**Expected Output**: Complete job recommendations with carousel format
**Validation**:
- Function returns success flag
- Jobs are relevant to candidate skills
- Carousel items have all required data
- Match percentages are accurate

## Troubleshooting

### Issue 1: API Returns 401 Unauthorized
**Cause**: Access token has expired or is invalid
**Solution**:
- Refresh the access token using the refresh token
- Verify token in configuration
- Check OAuth2 implementation from Step 1

### Issue 2: No Jobs Found
**Cause**: No job openings in the system or all jobs are filled
**Solution**:
- Check Zoho Recruit for active jobs
- Verify job status filter
- Ensure jobs have required_skills defined

### Issue 3: Match Percentage Too Low
**Cause**: Candidate skills don't match job requirements
**Solution**:
- Update candidate profile with correct skills
- Adjust minMatchPercentage threshold
- Add skill training recommendations

### Issue 4: Carousel Display Not Working
**Cause**: Format function not generating correct structure
**Solution**:
- Verify carousel item structure
- Check button action names
- Test with SalesIQ bot display

### Issue 5: Location Bonus Not Applied
**Cause**: Location field mismatch or null values
**Solution**:
- Ensure both candidate and job have valid locations
- Normalize location strings (trim, lowercase)
- Handle location abbreviations

## Integration Points

1. **Step 6 Dependency**: Uses Candidate ID from Step 6
2. **SalesIQ Bot Display**: Carousel results displayed in bot interface
3. **Step 8 Integration**: Selected jobs stored for tracking
4. **Zoho Recruit API**: Queries job and candidate data

## Future Enhancements

1. **Zia AI Integration**: Advanced skill matching using AI
2. **Experience Level Matching**: Consider years of experience
3. **Salary Expectations**: Filter by candidate salary range
4. **Remote Work Options**: Filter by remote/hybrid availability
5. **Department Preferences**: Consider preferred departments
6. **Notification System**: Alert candidates of matching jobs
7. **Analytics Dashboard**: Track job recommendation success rate

## Security Considerations

- Store API tokens securely (use encryption for refresh token)
- Validate all input parameters before API calls
- Log sensitive operations for audit trails
- Implement rate limiting for API calls
- Add request timeout handling (30 seconds)
- Validate API responses for data integrity

## Performance Optimization

- Cache job listings (refresh every 24 hours)
- Implement pagination for large result sets
- Use indexed searches for skill matching
- Parallel processing for multiple candidates
- CDN for static job descriptions

## Monitoring

- Log all job search requests
- Track match percentage statistics
- Monitor API response times
- Alert on repeated 401 errors
- Track failed searches for improvement

## Commit Information
- Date: 2025-01-16
- Author: Zoho Cliqtrix 2025 Team
- Status: Implementation Complete for Step 7
