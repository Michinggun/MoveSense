# Move Sense
Easy to use event detection based on user's activity(Walk, Running, On Vehicle), location, Nearby Places, & Weather.
Move Sense goal is to make it easy to use for everyone without complicated setup.

## API
Google Awareness API

## Feature
### MoveSenseSnapshot
How To Use:
Please Generate Your API Key and include it in manifest.xml

```xml
<meta-data
    android:name="com.google.android.awareness.API_KEY"
    android:value="[API_KEY]" />
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="[API_KEY]" />
```

```java
// Init Google API Client
moveSenseSnapshot = new MoveSenseSnapshot(this);
```

Callback Method:
```java
// Get User's Activity (Still, Walking, Running, On Vehicle)
moveSenseSnapshot.getDetectedActivity(new MoveSenseSnapshot.OnActivityDetectedListener() {
    @Override
    public void onActivityNotDetected(String message) {
        Toast.makeText(MainActivity.this, message, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onActivityDetected(ActivityRecognitionResult result) {
        DetectedActivity probableActivity = result.getMostProbableActivity();
        tvActivity.setText(MoveSenseHelper.getActivityType(probableActivity.getType()) + ": " + probableActivity.getConfidence());
    }
});

// Get User's Location
moveSenseSnapshot.getLocation(new MoveSenseSnapshot.OnLocationDetectedListener() {
    @Override
    public void onLocationNotDetected(String message) {
        Toast.makeText(MainActivity.this, message, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onLocationDetected(LocationResult result) {
        Location location = result.getLocation();
        tvLocation.setText("Lat: " + location.getLatitude() + ", Lng: " + location.getLongitude());
    }
});

// Get Nearby Places
moveSenseSnapshot.getNearbyPlaces(new MoveSenseSnapshot.OnNearbyPlacesDetectedListener() {
    @Override
    public void onNearbyPlacesNotDetected(String message) {
        Toast.makeText(MainActivity.this, message, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onNearbyPlacesDetected(PlacesResult result) {
        List<PlaceLikelihood> placeLikelihoodList = result.getPlaceLikelihoods();
        // Show the top 5 possible location results.
        String temp = "";
        if (placeLikelihoodList != null) {
            for (PlaceLikelihood placeLikelihood : placeLikelihoodList) {
                temp = temp + "\n" + placeLikelihood.getPlace().getName().toString() + ", likelihood: " + placeLikelihood.getLikelihood();
            }
            tvPlaces.setText(temp);
        }
    }
});

// Get Weathers (Rainy, Cloudy, Clear, etc)
moveSenseSnapshot.getWeathers(new MoveSenseSnapshot.OnWeatherDetectedListener() {
    @Override
    public void onWeatherNotDetected(String message) {
        Toast.makeText(MainActivity.this, message, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onWeatherDetected(WeatherResult result) {
        Weather weather = result.getWeather();
        tvWeather.setText("Weather: " + weather.getTemperature(Weather.CELSIUS) + "'C\n"
                + "Temperature feels like: " + weather.getFeelsLikeTemperature(Weather.CELSIUS) + "'C\n"
                + "Weather conditions: " + MoveSenseHelper.getWeatherConditions(weather.getConditions())
        );
    }
});
```

### MoveSenseFences
How To Use:
```java
// Init Google API Client
mMoveSenseFences = new MoveSenseFences(this);

// Listener to receive result of fencings
mMoveSenseReceiver = new MoveSenseReceiver();
mMoveSenseReceiver.setFenceListener(this);
registerReceiver(mMoveSenseReceiver, new IntentFilter(MoveSenseReceiver.FENCE_RECEIVER_ACTION));

// Add Fencing
mMoveSenseFences.addDetectActivityFence(MOVE_FENCE, DetectedActivityFence.ON_FOOT);
mMoveSenseFences.addHeadphoneFence(HEADPHONE_FENCE, HeadphoneState.PLUGGED_IN);
mMoveSenseFences.addAnyFence(NIGHT_FENCE, TimeFence.inTimeInterval(TimeFence.TIME_INTERVAL_NIGHT));

// Register the fence to receive callbacks.
mMoveSenseFences.registerFences();

// Unregister fences
mMoveSenseFences.unregisterFences(MOVE_FENCE, HEADPHONE_FENCE, NIGHT_FENCE);

// Remove listener
unregisterReceiver(mMoveSenseReceiver);

// Query Latest Data of Fences
mMoveSenseFences.queryFence(this, MOVE_FENCE, HEADPHONE_FENCE, NIGHT_FENCE);
```

Callback:
```java
// Handle Fencing State
@Override
public void onFenceDetected(FenceState state) {
    // Handle Action when fencing is met
    switch (state.getFenceKey()) {
        case MOVE_FENCE:
            tvMove.setText(state.getPreviousState() + ":" + state.getCurrentState());
            break;
        case HEADPHONE_FENCE:
            tvHeadphone.setText(state.getPreviousState() + ":" + state.getCurrentState());
            break;
        case NIGHT_FENCE:
            tvTime.setText(state.getPreviousState() + ":" + state.getCurrentState());
            break;
    }
}

@Override
public void onFenceNotDetected(FenceState state) {
    // Handle Action when fencing is not met
    switch (state.getFenceKey()) {
        case MOVE_FENCE:
            tvMove.setText("Not Detected");
            break;
        case HEADPHONE_FENCE:
            tvHeadphone.setText("Not Detected");
            break;
        case NIGHT_FENCE:
            tvTime.setText("Not Detected");
            break;
    }
}

// Handle Query Data Result
@Override
public void onQueryReceived(FenceQueryResult fenceQueryResult) {
    FenceStateMap map = fenceQueryResult.getFenceStateMap();
    for (String fenceKey : map.getFenceKeys()) {
        FenceState fenceState = map.getFenceState(fenceKey);
        switch (fenceKey) {
            case MOVE_FENCE:
                tvMove.setText(fenceState.getPreviousState() + ":" + fenceState.getCurrentState() + "\n" + DATE_FORMAT.format(
                        String.valueOf(new Date(fenceState.getLastFenceUpdateTimeMillis()))));
                break;
            case HEADPHONE_FENCE:
                tvHeadphone.setText(fenceState.getPreviousState() + ":" + fenceState.getCurrentState() + "\n" + DATE_FORMAT.format(
                        String.valueOf(new Date(fenceState.getLastFenceUpdateTimeMillis()))));
                break;
            case NIGHT_FENCE:
                tvTime.setText(fenceState.getPreviousState() + ":" + fenceState.getCurrentState() + "\n" + DATE_FORMAT.format(
                        String.valueOf(new Date(fenceState.getLastFenceUpdateTimeMillis()))));
                break;
        }
    }
}

@Override
public void onQueryNotReceived() {
    Toast.makeText(this, "Query Failed", Toast.LENGTH_SHORT).show();
}
```
## License

MIT License

Copyright (c) 2017 GARUDA WISNU KENCANA

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
