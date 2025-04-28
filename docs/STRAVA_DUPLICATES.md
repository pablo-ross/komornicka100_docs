# Duplicate Activity Detection Enhancement

## Overview

This document explains the enhancement to the Komornicka 100 application that adds intelligent duplicate activity detection and handling. This feature addresses the scenario where Strava users may track the same ride with multiple devices or accidentally upload the same activity more than once.

## Problem Statement

Users participating in the Komornicka 100 may occasionally create duplicate activities in Strava for the same actual ride due to:

1. Using multiple tracking devices simultaneously (e.g., a phone and a dedicated GPS device)
2. Manually uploading a ride that was also automatically synced
3. Technical issues causing an activity to be uploaded twice

Without proper handling, these duplicate activities could be counted multiple times in the contest, giving an unfair advantage.

## Solution

We've implemented a duplicate detection system that:

1. Identifies potential duplicate activities for the same user within a short time window
2. Compares key metrics like distance to confirm they're likely the same ride
3. When duplicates are detected, selects the activity with the highest route similarity score
4. Records all duplicate detection decisions in the database for auditing

### Key Features

#### 1. Time-Based Detection

Activities within a 4-hour window (±2 hours from the activity start time) are considered potential duplicates if they match other criteria.

#### 2. Distance Comparison

Activities with distances within 10% of each other are flagged as potential duplicates.

#### 3. Smart Selection Based on Quality

When duplicates are detected:
- If the new activity has a lower similarity score than an existing verified activity, it's rejected as a duplicate
- If the new activity has a higher similarity score, it replaces the existing activity in the contest

#### 4. Comprehensive Logging

All duplicate detection decisions are fully logged with:
- IDs of both activities involved
- Similarity scores of both activities
- Decision reason (rejected or replaced)

### Implementation Details

The duplicate detection logic is implemented in the `worker/verification.py` module through:

1. **`is_potential_duplicate`** function that:
   - Takes an activity and checks if it's a potential duplicate of already processed activities
   - Returns a tuple of (is_duplicate, duplicate_id, duplicate_similarity_score)

2. Modified **verification functions** that:
   - Check for duplicates before and after calculating similarity scores
   - Handle the replacement logic when a higher-quality duplicate is detected
   - Record all verification attempts and decisions

## How It Works

### Workflow

1. A new Strava activity is detected for a contest participant
2. The activity is initially checked against existing activities for potential duplication
3. The activity is verified against the source GPX routes to calculate its similarity score
4. With the similarity score known, duplicate detection is run again
5. If it's a duplicate with a higher score, the system replaces the existing activity
6. If it's a duplicate with a lower or equal score, it's rejected as a duplicate
7. The decision is recorded in the `activity_attempts` table

### Detection Logic

```python
# Simplified version of the duplicate detection algorithm
def is_potential_duplicate(user_id, activity_id, activity_details, similarity_score=None):
    # Get activity start time
    activity_start = parse_date(activity_details["start_date"])
    
    # Define time window (±2 hours)
    start_window = activity_start - timedelta(hours=2)
    end_window = activity_start + timedelta(hours=2)
    
    # Find potential duplicates within time window
    potential_duplicates = find_activities_in_time_window(
        user_id, activity_id, start_window, end_window
    )
    
    # Check distance similarity
    activity_distance = activity_details["distance"]
    
    for duplicate in potential_duplicates:
        # Calculate distance difference percentage
        distance_diff_percent = abs(duplicate.distance - activity_distance) / max(duplicate.distance, activity_distance)
        
        # If within 10% distance threshold
        if distance_diff_percent < 0.1:
            # Compare similarity scores if available
            if similarity_score is not None and similarity_score > duplicate.similarity_score:
                # Current activity has better score - should replace existing
                return True, duplicate.id, duplicate.similarity_score
            
            # Existing activity is fine or has better score
            return True, duplicate.id, duplicate.similarity_score
    
    # No duplicates found
    return False, None, None
```

## Example Scenarios

### Scenario 1: First Activity

When the first activity for a route is processed:
1. No duplicates are found
2. Similarity score is calculated against the source GPX
3. If the similarity score meets the threshold, the activity is verified and stored

### Scenario 2: Duplicate with Lower Quality

When a duplicate activity with a lower similarity score is detected:
1. Initial duplicate check finds a potential match based on time and distance
2. Verification still runs to calculate the similarity score
3. Second duplicate check confirms it's a duplicate with a lower score
4. The activity is marked as a duplicate in `activity_attempts` but not added to verified activities
5. The user is not notified, and the leaderboard remains unchanged

### Scenario 3: Duplicate with Higher Quality

When a duplicate activity with a higher similarity score is detected:
1. Initial duplicate check finds a potential match
2. Verification runs to calculate the similarity score
3. Second duplicate check confirms it's a duplicate with a higher score
4. The existing activity is removed from the database
5. The new activity is added as a verified activity
6. Leaderboard remains unchanged (count stays the same)

## Testing

A comprehensive test script (`test_duplicate_detection.py`) has been created to verify the duplicate detection logic. This script:

1. Sets up an in-memory test database
2. Creates test data (user, source GPX)
3. Simulates multiple activities with varying similarity scores
4. Verifies that the duplicate detection logic works as expected

To run the test script:

```bash
python test_duplicate_detection.py
```

## Configuration Parameters

The duplicate detection system has the following configurable parameters:

1. **Time Window**: How much time before and after an activity to look for potential duplicates
   - Default: ±2 hours (4-hour window)
   - Can be adjusted if needed for specific contest requirements

2. **Distance Threshold**: Maximum distance difference percentage to consider activities as duplicates
   - Default: 10%
   - Lower values make the detection more strict
   - Higher values make it more lenient

These parameters can be adjusted in the `is_potential_duplicate` function in `worker/verification.py`.

## Benefits of This Approach

1. **Fairness**: Ensures each actual ride is counted only once in the contest
2. **Quality Optimization**: Always keeps the activity with the best route matching
3. **Transparency**: Full logging of duplicate detection decisions
4. **Flexibility**: Configurable parameters to adjust sensitivity

## Future Improvements

Potential enhancements for the duplicate detection system:

1. **User Notification**: Inform users when a duplicate activity is detected
2. **Admin Interface**: Create an admin view to review and override duplicate decisions
3. **Machine Learning**: Implement more sophisticated detection using additional features
4. **Performance Optimization**: Add database indexes to improve query performance for large datasets

## Conclusion

The duplicate activity detection enhancement ensures that the Komornicka 100 counts each unique ride only once, even when users accidentally or intentionally create multiple Strava activities for the same ride. By always selecting the activity with the highest quality match to the source route, we maintain fair competition while optimizing the accuracy of route verification.