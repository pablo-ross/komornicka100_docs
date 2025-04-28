# GPX Route Verification System

This document explains how the GPX route verification system works in the Komornicka 100 application, including implementation details, configuration options, anti-fraud measures, and troubleshooting guidance.

## Overview

The GPX verification system automatically verifies Strava activities against predefined routes by:

1. Retrieving activity data from the Strava API
2. Comparing activity GPS coordinates with source GPX routes
3. Calculating a similarity score to determine if the activity matches a route
4. Applying anti-fraud measures to detect suspicious activities
5. Storing verified activities in the database and updating the leaderboard

## Verification Algorithm

### Core Verification Logic

The verification process follows these steps:

1. **Data Retrieval**:
   - Fetch activity details and GPS streams from the Strava API
   - Load source GPX files from the filesystem

2. **Route Comparison**:
   - Convert activity streams and source GPX to coordinate points
   - Simplify both sets of points using the Douglas-Peucker algorithm to improve performance
   - Create a LineString from source points using the Shapely library

3. **Similarity Calculation**:
   - For each point in the activity, calculate the distance to the closest point on the source route
   - Count points within the maximum deviation threshold (configurable, default: 20 meters)
   - Calculate similarity score as percentage of points within threshold
   - Apply anti-fraud checks to detect suspicious patterns

4. **Verification Criteria**:
   - Activity must exceed minimum distance (configurable, default: 95 km)
   - Similarity score must exceed threshold (configurable, default: 80%)
   - Activity must not exceed maximum duration (configurable, default: 24 hours)
   - Activity must not trigger anti-fraud detection rules
   - Activity must not be a duplicate of an already verified activity

### Implementation Details

The key components of the verification system are:

- **`services.py`**: Contains functions for Strava API interaction and GPS calculations
- **`verification/`**: Package containing modular verification components:
  - `activity.py`: Main verification entry point
  - `sources.py`: Logic for matching activities against source routes
  - `duplicates.py`: Duplicate detection system
  - `time_check.py`: Activity duration verification

The main verification function is `verify_activity_against_source` which compares an activity against a source route:

```python
def verify_activity_against_source(
    source_gpx_content: str,
    activity_points: List[Tuple[float, float]],
    activity_distance: float
) -> Dict[str, any]:
    # Check minimum distance requirement
    required_distance = settings.MIN_ACTIVITY_DISTANCE_KM * 1000  # Convert to meters
    if activity_distance < required_distance:
        return {
            "verified": False,
            "similarity_score": 0.0,
            "message": f"Activity distance ({activity_distance/1000:.1f}km) is less than required ({settings.MIN_ACTIVITY_DISTANCE_KM}km)"
        }
    
    # Load and process source route points
    source_points = load_gpx_points(source_gpx_content)
    
    # Calculate similarity between routes with anti-fraud checks
    similarity_score, deviations, stats = calculate_similarity(
        source_points,
        activity_points,
        max_deviation=settings.GPS_MAX_DEVIATION_METERS,
        min_avg_deviation=min_avg_deviation,
        min_std_deviation=min_std_deviation
    )
    
    # Determine if activity should be verified
    verified = meets_similarity and not is_suspicious
    
    return {
        "verified": verified,
        "similarity_score": similarity_score,
        "message": message,
        "stats": stats,
        "is_suspicious": is_suspicious
    }
```

## Anti-Fraud Measures

The system includes several anti-fraud measures to detect and prevent cheating:

### Statistical Analysis

To catch potential fraud (like uploading the original GPX file), the system analyzes:

1. **Average Deviation**: The mean distance between activity points and source route
   - Suspiciously low if below `MIN_AVG_DEVIATION_METERS` (default: 2.0 meters)
   
2. **Standard Deviation**: The variation in deviation distances
   - Suspiciously low if below `MIN_STD_DEVIATION_METERS` (default: 1.0 meters)
   
3. **Near-Zero Deviations**: The percentage of points with near-zero deviation
   - Suspicious if above `SUSPICIOUS_ZERO_PERCENT` (default: 30%)

Natural GPS recordings will always have some variation due to GPS accuracy, different riding lines, and device sampling rates. Activities with unnaturally perfect matches are flagged as suspicious.

### Duplicate Detection

The system prevents users from getting credit for the same ride multiple times:

1. **Time-Based Detection**: Activities within a 4-hour window (±2 hours) are checked
2. **Distance Comparison**: Activities with distances within 10% are potential duplicates
3. **Quality Selection**: When duplicates are detected, the one with the higher similarity score is kept
4. **Record Keeping**: All duplicate decisions are logged for auditing

Implementation in `duplicates.py`:

```python
async def is_potential_duplicate(
    db: Session,
    user_id: str,
    activity_id: str,
    activity_details: Dict[str, any],
    similarity_score: Optional[float] = None
) -> Tuple[bool, Optional[str], Optional[float]]:
    # Extract start_date from the activity
    activity_start = datetime.fromisoformat(activity_details.get("start_date").replace("Z", "+00:00"))
    
    # Set time window for duplicate detection (2 hours before and after)
    start_window = activity_start - timedelta(hours=2)
    end_window = activity_start + timedelta(hours=2)
    
    # Query for activities from the same user in the same time window
    potential_duplicates = db.query(Activity).filter(
        Activity.user_id == user_id,
        Activity.strava_activity_id != activity_id,
        Activity.start_date >= start_window,
        Activity.start_date <= end_window
    ).all()
    
    # Check distance similarity with each potential duplicate
    activity_distance = activity_details.get("distance", 0)
    
    for duplicate in potential_duplicates:
        distance_diff_percent = abs(duplicate.distance - activity_distance) / max(duplicate.distance, activity_distance)
        
        # If the distance is within 10% of an existing activity, consider it a duplicate
        if distance_diff_percent < 0.1:  # 10% threshold
            # Compare similarity scores if available
            if similarity_score is not None and similarity_score > duplicate.similarity_score:
                # Current activity has a better score - consider replacing
                return True, duplicate.strava_activity_id, duplicate.similarity_score
            
            # Existing activity is fine or has better score
            return True, duplicate.strava_activity_id, duplicate.similarity_score
    
    return False, None, None
```

## Configuration

### Main Verification Parameters

The verification system is configured through environment variables in `.env`:

```
# GPX verification settings
ROUTE_SIMILARITY_THRESHOLD=0.8     # 80% similarity required
GPS_MAX_DEVIATION_METERS=35.0      # 35 meters maximum deviation
MIN_ACTIVITY_DISTANCE_KM=98.0      # 98 kilometers minimum
MAX_ACTIVITY_DURATION=24.0         # 24 hours maximum duration
```

These parameters affect:

- **`ROUTE_SIMILARITY_THRESHOLD`**: Minimum percentage of points that must be within `GPS_MAX_DEVIATION_METERS` of the source route
- **`GPS_MAX_DEVIATION_METERS`**: Maximum allowed distance between activity points and source route
- **`MIN_ACTIVITY_DISTANCE_KM`**: Minimum activity distance required
- **`MAX_ACTIVITY_DURATION`**: Maximum allowed activity duration in hours

### Anti-Fraud Parameters

The anti-fraud settings are defined in `settings.py`:

```python
# Anti-fraud settings in settings.py
MIN_AVG_DEVIATION_METERS = 2.0  # Minimum average deviation in meters
MIN_STD_DEVIATION_METERS = 1.0  # Minimum standard deviation in meters
SUSPICIOUS_ZERO_PERCENT = 0.3   # 30% of points with near-zero deviation is suspicious
```

## Source GPX Management

### Adding Source Routes

Source GPX files should be placed in the `gpx/` directory. The system automatically detects and loads them:

1. **Automatic Detection**: The system scans the `gpx/` directory for `.gpx` files
2. **Database Entry**: Each file is added to the `source_gpxs` table with metadata
3. **File Properties**: The system extracts route name, description, and distance

To add new routes:

1. Place your `.gpx` file in the `gpx/` directory
2. Run the initialization script:
   ```bash
   ./.scripts/init-gpx-files.sh
   ```
   or directly:
   ```bash
   docker compose exec api python scripts/init_source_gpx.py
   ```

### Source Route Requirements

For optimal verification results:

1. **File Format**: Standard GPX format with track segments
2. **Route Quality**: Clean routes without excessive points or gaps
3. **Naming**: Use descriptive names within the GPX file

## Testing and Fine-Tuning

### Testing Verification

A dedicated script is available for testing the Strava checker functionality:

```bash
# Run from project root
docker compose exec worker python scripts/run_strava_checker.py

# Test only activity verification
docker compose exec worker python scripts/run_strava_checker.py --activities-only

# Test only user connections
docker compose exec worker python scripts/run_strava_checker.py --users-only
```

### Adjusting Parameters

To fine-tune the verification system:

1. **Similarity Threshold**:
   - **Higher** (e.g., 0.9): More strict, fewer false positives, may miss legitimate activities
   - **Lower** (e.g., 0.7): More lenient, fewer false negatives, might accept partial matches

2. **Maximum Deviation**:
   - **Higher** (e.g., 30m): More forgiving of GPS inaccuracies, works better in urban areas
   - **Lower** (e.g., 10m): Stricter matching, best for open areas with good GPS signal

3. **Anti-Fraud Settings**:
   - Adjust based on observations of legitimate activities and potential fraud attempts
   - Higher thresholds may be needed for activities recorded on specific devices

### Monitoring and Analysis

The system records detailed information about verification attempts:

1. **Activity Attempts**: Every verification attempt is recorded in the `activity_attempts` table
2. **Statistics**: For verified activities, additional statistics are stored
3. **Logs**: Detailed logs are available in worker logs

To analyze verification results:

```bash
# View worker logs
docker compose logs -f worker

# List all verification attempts
docker compose exec db psql -U <user> -d <database> -c "SELECT * FROM activity_attempts ORDER BY checked_at DESC LIMIT 10;"
```

## Handling Edge Cases

### GPS Inaccuracies

The system is designed to handle common GPS issues:

1. **Signal Loss**: Short gaps in GPS tracking are tolerated
2. **Urban Canyons**: Higher deviation allowance helps with poor signal areas
3. **Device Variations**: Statistical analysis accounts for different device behaviors

### Route Variations

To handle different ways people might ride the same route:

1. **Route Direction**: The current algorithm is bidirectional (can ride in either direction)
2. **Detours**: Small detours might still verify if enough points match
3. **Starting Points**: Riders can start anywhere on the route

## Troubleshooting

### Common Issues

1. **Activities Not Verifying**:
   - Ensure activity follows the source route closely
   - Check if activity meets minimum distance requirement
   - Verify that the GPX file is properly formatted
   - Check worker logs for specific failure messages

2. **False Negatives** (valid activities not verifying):
   - The activity may deviate too much from the source route
   - GPS signal issues may have affected tracking quality
   - Try decreasing the similarity threshold or increasing max deviation

3. **False Positives** (invalid activities verifying):
   - The similarity threshold may be too low
   - The max deviation may be too high
   - Consider adjusting anti-fraud parameters

4. **Duplicate Detection Issues**:
   - Check time window configuration
   - Verify distance threshold for duplicates
   - Inspect activity start times

### Debugging Methods

For advanced debugging:

1. **Direct API Testing**:
   ```python
   # Example verification test with explicit values
   gpx_content = open('gpx/route.gpx', 'r').read()
   activity_points = [(lat1, lon1), (lat2, lon2), ...] # From Strava
   result = verify_activity_against_source(gpx_content, activity_points, distance_meters)
   print(result)
   ```

2. **Visualizing Routes**:
   - Use the frontend's map component to visualize the activity and source route
   - Extract and compare routes using GIS tools
   - Use the binary `streams` data stored in the database for detailed analysis

## Performance Considerations

For optimal performance:

1. **Point Simplification**: The system uses the Douglas-Peucker algorithm to reduce points
2. **Database Indexes**: Important fields are indexed for faster queries
3. **Worker Processes**: Multi-process worker configuration is available for high load
4. **Activity Stream Compression**: Activity GPS data is compressed for efficient storage

## Advanced Topics

### Custom Verification Logic

To implement custom verification rules:

1. Create a new module in the `worker/verification/` package
2. Add your custom verification logic
3. Update the main verification flow in `activity.py` or `sources.py`

### Extending Anti-Fraud Measures

Additional anti-fraud measures could include:

1. **Speed Analysis**: Detect unrealistic speeds
2. **Pattern Recognition**: Identify artificially generated GPS data
3. **User Behavior Analysis**: Flag users with suspicious pattern of activities
4. **Time-of-Day Verification**: Check if activities occur during reasonable hours

These can be implemented in new modules within the verification package.

## Glossary

- **Source GPX**: Predefined route that users need to match
- **Activity**: A bike ride recorded in Strava
- **Similarity Score**: Percentage of points in an activity that match the source route
- **Deviation**: Distance between an activity point and the closest point on the source route
- **Anti-Fraud Measures**: Checks to detect and prevent cheating
- **Duplicate Detection**: System to prevent double-counting the same ride