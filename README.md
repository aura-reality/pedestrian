# pedestrian
a scavenger hunt


## Pseudocode for routing and rendering

The following code is memory-less, and computes given a single frame. Of course, its detections (of i.e. sidewalks, roads, and north) would be improved by computing over the entire session, as opposed to a single frame.

```swift

// The main routing loop. 
// Runs throughout the duration of the routing session.
func route(query) {
    let destination = geocode(query)

    while true {
        let loc = getCurrentLocation()
        if distance(loc, destination) < threshold { 
            break
        } else {
            let route = findRoute(loc, destination)
            render(loc, route)        
        }
    }
} 

// Renders the computed route given a user's location
func render(currentLocation, route) {
    let indicators = getDirectionIndicators(currentLocation, route)
    if indicators.hasLine {
        let maskedHeadingLine = maybeObfuscate(indicators.line, getDetectedPlanes())
        iOS.draw(maskedHeadingLine)
    } else if indicators.hasArrow {
        iOS.draw(nearbyDirection.arrow) 
    }
}

// Computes the navigation indicators for the display
func getDirectionIndicators(currentLocation, route) {
    
    // detect things and then set their orientation for map matching
    let detections = visionDetectMapFeatures()
    let north = guessNorth(detections)
    detections.setNorth(north)
    detections.setCurrentLocation(currentLocation)
    
    let actualDirection = route.getHeadingDirection()
    
    let parallelSidewalk = detections.sidewalks.filter(actualDirection)
    if !parallelSidewalk.isEmpty {
        return parallelSidewalk.line
    } else {
        // no matching sidewalk, so just show arrows
        return arrowForDirection(north, actualDirection)        
    }
} 

func guessNorth(detections, map) {
    // detect from sidewalks or roads or street signs etc...
}

func findRoute(positionA, positionB) {
    // use OSM for pedestrian routing
}

func maybeObfuscate(shape, planes) {
    // returns the given shape potentially obfuscated by the given planes
}
```

### Given the above, the major problems are:

* Geocoding a query
* Routing b/w two points
* Obfuscation (of a line)
  * good (distant) plane detection
* Detecting map features
  * sidewalks
  * streets
  * street-signs
* Detecting north (likely by anaylzing the detected map features)

#### In the beginning (AKA prototype) we could assume the following hard things are given:
* geocoding
* route
* north

Then, we would just draw hovering arrows pointing in the correct direction.
