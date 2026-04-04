# Agent Skills Best Practices for Omni-X

This document outlines best practices for implementing and using Omni-X Twitter Skills, based on Claude's agent skills guidelines and industry standards.

---

## Table of Contents

1. [Skill Design Principles](#skill-design-principles)
2. [Input Validation](#input-validation)
3. [Error Handling](#error-handling)
4. [Response Format](#response-format)
5. [Performance Optimization](#performance-optimization)
6. [Security Considerations](#security-considerations)
7. [Testing and Reliability](#testing-and-reliability)
8. [Documentation Standards](#documentation-standards)

---

## Skill Design Principles

### 1. Single Responsibility

Each skill should do one thing well. Omni-X follows this principle:

```python
# ✓ GOOD: Each skill has a single, clear purpose
get_user_profile()      # Only gets profile data
get_user_tweets()       # Only gets tweets
get_user_followers()    # Only gets followers

# ✗ BAD: Don't combine multiple operations
get_user_everything()   # Too broad, unclear purpose
```

### 2. Predictable Behavior

Skills should behave consistently and predictably:

```python
# ✓ GOOD: Consistent parameter names and types
interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "elonmusk", "count": 10}
)

# ✓ GOOD: Consistent response structure
{
    "success": True,
    "data": [...],
    "count": 10,
    "skill_name": "get_user_tweets",
    "parameters": {...}
}
```

### 3. Composability

Skills should be composable and work well together:

```python
# ✓ GOOD: Skills can be chained
profile = interface.execute_skill("get_user_profile", {"username": "elonmusk"})
if profile["success"]:
    tweets = interface.execute_skill("get_user_tweets", {"username": "elonmusk"})
    followers = interface.execute_skill("get_user_followers", {"username": "elonmusk"})
```

### 4. Discoverability

Skills should be easily discoverable:

```python
# ✓ GOOD: Dynamic skill discovery
skills = interface.get_available_skills()
for skill_name, skill_info in skills.items():
    print(f"{skill_name}: {skill_info['description']}")
    print(f"Parameters: {skill_info['parameters']}")
```

---

## Input Validation

### 1. Validate All Inputs

Always validate input parameters before processing:

```python
# ✓ GOOD: Validate username format
def validate_username(username: str) -> bool:
    if not username or not isinstance(username, str):
        return False
    if username.startswith('@'):
        return False  # Username should not include @
    if len(username) > 15:  # Twitter username max length
        return False
    return True

# Usage in skill
if not validate_username(username):
    return {
        "success": False,
        "error": "Invalid username format. Username should not include @ and must be 1-15 characters."
    }
```

### 2. Provide Clear Error Messages

```python
# ✓ GOOD: Specific, actionable error messages
{
    "success": False,
    "error": "Username 'elonmusk' not found. Please check the username and try again.",
    "error_code": "USER_NOT_FOUND"
}

# ✗ BAD: Vague error messages
{
    "success": False,
    "error": "Error occurred"
}
```

### 3. Set Reasonable Defaults

```python
# ✓ GOOD: Sensible defaults for optional parameters
def get_user_tweets(username: str, count: int = 10):
    # Default count is reasonable for most use cases
    if count < 1:
        count = 1
    if count > 100:
        count = 100  # Prevent excessive requests
    # ...
```

### 4. Type Checking

```python
# ✓ GOOD: Validate parameter types
def execute_skill(self, skill_name: str, parameters: dict):
    if not isinstance(skill_name, str):
        return {
            "success": False,
            "error": "skill_name must be a string"
        }
    
    if not isinstance(parameters, dict):
        return {
            "success": False,
            "error": "parameters must be a dictionary"
        }
```

---

## Error Handling

### 1. Graceful Degradation

Handle errors gracefully without crashing:

```python
# ✓ GOOD: Catch and handle exceptions
def get_user_profile(self, username: str):
    try:
        profile = self.twitter.get_user_info(username)
        return {
            "success": True,
            "data": profile
        }
    except UserNotFound:
        return {
            "success": False,
            "error": f"User '{username}' not found",
            "error_code": "USER_NOT_FOUND"
        }
    except RateLimitExceeded:
        return {
            "success": False,
            "error": "Rate limit exceeded. Please try again later.",
            "error_code": "RATE_LIMIT",
            "retry_after": 60
        }
    except Exception as e:
        return {
            "success": False,
            "error": f"Unexpected error: {str(e)}",
            "error_code": "UNKNOWN_ERROR"
        }
```

### 2. Provide Context

Include relevant context in error responses:

```python
# ✓ GOOD: Include context for debugging
{
    "success": False,
    "error": "Authentication required for this operation",
    "error_code": "AUTH_REQUIRED",
    "skill_name": "get_user_followers",
    "parameters": {"username": "elonmusk"},
    "suggestion": "Please provide an auth_token to access this feature"
}
```

### 3. Error Recovery Suggestions

```python
# ✓ GOOD: Suggest recovery actions
{
    "success": False,
    "error": "Unknown skill: get_user_stats",
    "error_code": "UNKNOWN_SKILL",
    "available_skills": ["get_user_profile", "get_user_tweets", ...],
    "suggestion": "Did you mean 'get_user_profile'?"
}
```

---

## Response Format

### 1. Consistent Structure

All skills return the same base structure:

```python
# ✓ GOOD: Consistent response format
{
    "success": bool,           # Always present
    "data": any,              # Present on success
    "error": str,             # Present on failure
    "error_code": str,        # Present on failure
    "skill_name": str,        # Always present
    "parameters": dict,       # Always present
    "count": int,             # Optional: number of items
    "has_next_page": bool,    # Optional: pagination info
    "cursor": str,            # Optional: pagination cursor
    "metadata": dict          # Optional: additional info
}
```

### 2. Include Metadata

Provide useful metadata for AI agents:

```python
# ✓ GOOD: Include execution metadata
{
    "success": True,
    "data": [...],
    "count": 10,
    "skill_name": "get_user_tweets",
    "parameters": {"username": "elonmusk", "count": 10},
    "metadata": {
        "execution_time_ms": 1234,
        "api_version": "1.0",
        "rate_limit_remaining": 45,
        "timestamp": "2026-04-05T01:30:00Z"
    }
}
```

### 3. Pagination Support

For large datasets, support pagination:

```python
# ✓ GOOD: Include pagination information
{
    "success": True,
    "data": [...],
    "count": 20,
    "has_next_page": True,
    "cursor": "DAABCgABGVl...",
    "total_available": 1500,  # If known
    "page_info": {
        "current_page": 1,
        "items_per_page": 20
    }
}
```

---

## Performance Optimization

### 1. Implement Caching

Cache frequently accessed data:

```python
# ✓ GOOD: Cache user profiles
from functools import lru_cache
from datetime import datetime, timedelta

class TwitterSkills:
    def __init__(self):
        self._cache = {}
        self._cache_ttl = timedelta(minutes=5)
    
    def get_user_profile(self, username: str):
        cache_key = f"profile:{username}"
        
        # Check cache
        if cache_key in self._cache:
            cached_data, timestamp = self._cache[cache_key]
            if datetime.now() - timestamp < self._cache_ttl:
                return {
                    "success": True,
                    "data": cached_data,
                    "cached": True
                }
        
        # Fetch fresh data
        profile = self.twitter.get_user_info(username)
        self._cache[cache_key] = (profile, datetime.now())
        
        return {
            "success": True,
            "data": profile,
            "cached": False
        }
```

### 2. Batch Operations

Support batch operations when possible:

```python
# ✓ GOOD: Batch user profile retrieval
def get_multiple_profiles(self, usernames: list):
    results = []
    for username in usernames:
        result = self.get_user_profile(username)
        results.append(result)
    
    return {
        "success": True,
        "data": results,
        "count": len(results),
        "skill_name": "get_multiple_profiles",
        "parameters": {"usernames": usernames}
    }
```

### 3. Rate Limiting

Implement rate limiting to prevent abuse:

```python
# ✓ GOOD: Rate limiting with backoff
import time
from collections import deque

class RateLimiter:
    def __init__(self, max_requests: int, time_window: int):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = deque()
    
    def allow_request(self) -> bool:
        now = time.time()
        
        # Remove old requests outside time window
        while self.requests and self.requests[0] < now - self.time_window:
            self.requests.popleft()
        
        # Check if we can make a new request
        if len(self.requests) < self.max_requests:
            self.requests.append(now)
            return True
        
        return False
    
    def wait_time(self) -> float:
        if not self.requests:
            return 0
        oldest = self.requests[0]
        return max(0, self.time_window - (time.time() - oldest))
```

---

## Security Considerations

### 1. Secure Token Handling

Never log or expose authentication tokens:

```python
# ✓ GOOD: Mask sensitive data in logs
def log_request(self, skill_name: str, parameters: dict):
    safe_params = parameters.copy()
    if 'auth_token' in safe_params:
        safe_params['auth_token'] = '***REDACTED***'
    
    logger.info(f"Executing skill: {skill_name}, params: {safe_params}")

# ✗ BAD: Logging sensitive data
logger.info(f"Auth token: {auth_token}")  # Never do this!
```

### 2. Input Sanitization

Sanitize all user inputs:

```python
# ✓ GOOD: Sanitize inputs
import re

def sanitize_username(username: str) -> str:
    # Remove any potentially harmful characters
    username = re.sub(r'[^\w\-]', '', username)
    return username[:15]  # Enforce max length

def sanitize_query(query: str) -> str:
    # Remove script tags and other dangerous content
    query = re.sub(r'<script.*?</script>', '', query, flags=re.IGNORECASE)
    return query.strip()
```

### 3. Environment Variables for Secrets

Store sensitive data in environment variables:

```python
# ✓ GOOD: Use environment variables
import os

auth_token = os.getenv('TWITTER_AUTH_TOKEN')
if not auth_token:
    raise ValueError("TWITTER_AUTH_TOKEN environment variable not set")

interface = TwitterSkillInterface(auth_token=auth_token)

# ✗ BAD: Hardcoded secrets
auth_token = "abc123xyz"  # Never do this!
```

### 4. Validate Data Sources

Validate data from external sources:

```python
# ✓ GOOD: Validate API responses
def validate_tweet_data(tweet: dict) -> bool:
    required_fields = ['id', 'text', 'created_at', 'user']
    return all(field in tweet for field in required_fields)

def get_user_tweets(self, username: str):
    tweets = self.twitter.get_tweets(username)
    
    # Validate each tweet
    valid_tweets = [t for t in tweets if validate_tweet_data(t)]
    
    return {
        "success": True,
        "data": valid_tweets,
        "count": len(valid_tweets)
    }
```

---

## Testing and Reliability

### 1. Unit Tests

Write comprehensive unit tests:

```python
# ✓ GOOD: Test all skill functions
import unittest
from scripts import TwitterSkillInterface

class TestTwitterSkills(unittest.TestCase):
    def setUp(self):
        self.interface = TwitterSkillInterface()
    
    def test_get_user_profile_success(self):
        result = self.interface.execute_skill(
            skill_name="get_user_profile",
            parameters={"username": "elonmusk"}
        )
        self.assertTrue(result["success"])
        self.assertIn("data", result)
    
    def test_get_user_profile_invalid_username(self):
        result = self.interface.execute_skill(
            skill_name="get_user_profile",
            parameters={"username": ""}
        )
        self.assertFalse(result["success"])
        self.assertIn("error", result)
    
    def test_unknown_skill(self):
        result = self.interface.execute_skill(
            skill_name="invalid_skill",
            parameters={}
        )
        self.assertFalse(result["success"])
        self.assertIn("available_skills", result)
```

### 2. Integration Tests

Test skill interactions:

```python
# ✓ GOOD: Test skill workflows
def test_complete_workflow(self):
    # Get profile
    profile = self.interface.execute_skill(
        "get_user_profile",
        {"username": "elonmusk"}
    )
    self.assertTrue(profile["success"])
    
    # Get tweets
    tweets = self.interface.execute_skill(
        "get_user_tweets",
        {"username": "elonmusk", "count": 5}
    )
    self.assertTrue(tweets["success"])
    self.assertEqual(tweets["count"], 5)
```

### 3. Error Scenario Tests

Test error handling:

```python
# ✓ GOOD: Test error scenarios
def test_rate_limit_handling(self):
    # Simulate rate limit
    for i in range(100):
        result = self.interface.execute_skill(
            "get_user_profile",
            {"username": f"user{i}"}
        )
    
    # Should handle rate limit gracefully
    self.assertIn("error_code", result)
    if not result["success"]:
        self.assertEqual(result["error_code"], "RATE_LIMIT")
```

### 4. Mock External Dependencies

Use mocks for testing:

```python
# ✓ GOOD: Mock Twitter API
from unittest.mock import Mock, patch

def test_with_mock(self):
    with patch('tweeterpy.TweeterPy') as mock_twitter:
        mock_twitter.return_value.get_user_info.return_value = {
            "username": "elonmusk",
            "name": "Elon Musk"
        }
        
        result = self.interface.execute_skill(
            "get_user_profile",
            {"username": "elonmusk"}
        )
        
        self.assertTrue(result["success"])
        self.assertEqual(result["data"]["username"], "elonmusk")
```

---

## Documentation Standards

### 1. Clear Skill Descriptions

Each skill should have clear documentation:

```python
# ✓ GOOD: Comprehensive skill documentation
{
    "get_user_tweets": {
        "description": "Retrieve recent tweets from a specified user's timeline",
        "parameters": [
            {
                "name": "username",
                "type": "string",
                "required": True,
                "description": "Twitter username without @ symbol (e.g., 'elonmusk')",
                "example": "elonmusk"
            },
            {
                "name": "count",
                "type": "integer",
                "required": False,
                "default": 10,
                "description": "Number of tweets to retrieve (1-100)",
                "example": 20
            }
        ],
        "returns": {
            "success": "boolean - Whether the operation succeeded",
            "data": "array - List of tweet objects",
            "count": "integer - Number of tweets returned",
            "has_next_page": "boolean - Whether more tweets are available",
            "cursor": "string - Pagination cursor for next page"
        },
        "requires_auth": False,
        "rate_limit": "180 requests per 15 minutes",
        "example": {
            "request": {
                "skill_name": "get_user_tweets",
                "parameters": {"username": "elonmusk", "count": 5}
            },
            "response": {
                "success": True,
                "data": [...],
                "count": 5
            }
        }
    }
}
```

### 2. Usage Examples

Provide clear usage examples:

```python
# ✓ GOOD: Multiple usage examples
"""
Example 1: Basic usage
----------------------
result = interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "elonmusk"}
)

Example 2: With pagination
--------------------------
result = interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "elonmusk", "count": 20}
)

if result["has_next_page"]:
    next_result = interface.execute_skill(
        skill_name="get_user_tweets",
        parameters={
            "username": "elonmusk",
            "count": 20,
            "cursor": result["cursor"]
        }
    )

Example 3: Error handling
-------------------------
result = interface.execute_skill(
    skill_name="get_user_tweets",
    parameters={"username": "nonexistent_user"}
)

if not result["success"]:
    print(f"Error: {result['error']}")
    print(f"Error code: {result['error_code']}")
"""
```

### 3. Changelog

Maintain a changelog for skill updates:

```markdown
# Changelog

## [1.1.0] - 2026-04-05
### Added
- Added `get_user_media` skill for retrieving media content
- Added pagination support to all list-based skills
- Added rate limit information in response metadata

### Changed
- Improved error messages with specific error codes
- Updated `search_tweets` to support more filter options

### Fixed
- Fixed cursor handling in pagination
- Fixed authentication token validation

## [1.0.0] - 2026-03-01
### Added
- Initial release with 6 core skills
- Dynamic skill discovery
- Standardized response format
```

---

## Implementation Checklist

Use this checklist when implementing new skills:

- [ ] **Design**
  - [ ] Single, clear purpose
  - [ ] Consistent with existing skills
  - [ ] Composable with other skills

- [ ] **Input Validation**
  - [ ] All parameters validated
  - [ ] Type checking implemented
  - [ ] Reasonable defaults set
  - [ ] Clear error messages

- [ ] **Error Handling**
  - [ ] All exceptions caught
  - [ ] Graceful degradation
  - [ ] Error codes defined
  - [ ] Recovery suggestions provided

- [ ] **Response Format**
  - [ ] Consistent structure
  - [ ] Includes metadata
  - [ ] Pagination support (if applicable)

- [ ] **Performance**
  - [ ] Caching implemented (if applicable)
  - [ ] Rate limiting considered
  - [ ] Batch operations supported (if applicable)

- [ ] **Security**
  - [ ] Input sanitization
  - [ ] Token handling secure
  - [ ] No sensitive data in logs
  - [ ] Data validation

- [ ] **Testing**
  - [ ] Unit tests written
  - [ ] Integration tests written
  - [ ] Error scenarios tested
  - [ ] Mocks used appropriately

- [ ] **Documentation**
  - [ ] Clear description
  - [ ] Parameter documentation
  - [ ] Return value documentation
  - [ ] Usage examples
  - [ ] Error codes documented

---

## Conclusion

Following these best practices ensures that Omni-X Twitter Skills are:

- **Reliable**: Consistent behavior and error handling
- **Secure**: Proper input validation and token handling
- **Performant**: Optimized with caching and rate limiting
- **Maintainable**: Well-documented and tested
- **User-friendly**: Clear errors and helpful suggestions

For more information, see:
- [AI_AGENT_GUIDE.md](AI_AGENT_GUIDE.md) - Integration guide
- [AI_TESTING_GUIDE.md](AI_TESTING_GUIDE.md) - Testing guide
- [README.md](README.md) - General documentation
