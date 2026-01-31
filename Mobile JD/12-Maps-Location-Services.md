---
title: Maps & Location Services
date: 2026-01-31
tags:
  - maps
  - location
  - gps
  - arcgis
  - interview
---

# Maps & Location Services

> [!important] JD Optional Skills
> "Experience integrating Google Maps SDK, Mapbox, or location-based services"
> "Experience with ArcGIS Runtime SDK for map rendering, GIS data handling"

## Google Maps SDK

### Android Setup

```kotlin
// build.gradle
implementation("com.google.android.gms:play-services-maps:18.2.0")
implementation("com.google.android.gms:play-services-location:21.0.1")

// AndroidManifest.xml
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="${MAPS_API_KEY}" />
```

### Basic Map

```kotlin
class MapsActivity : AppCompatActivity(), OnMapReadyCallback {
    
    private lateinit var map: GoogleMap
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_maps)
        
        val mapFragment = supportFragmentManager
            .findFragmentById(R.id.map) as SupportMapFragment
        mapFragment.getMapAsync(this)
    }
    
    override fun onMapReady(googleMap: GoogleMap) {
        map = googleMap
        
        // Add marker
        val location = LatLng(20.2961, 85.8245) // Bhubaneswar
        map.addMarker(MarkerOptions().position(location).title("Office"))
        map.moveCamera(CameraUpdateFactory.newLatLngZoom(location, 15f))
    }
}
```

### iOS Setup

```swift
// AppDelegate
import GoogleMaps

func application(_ application: UIApplication, didFinishLaunchingWithOptions...) {
    GMSServices.provideAPIKey("YOUR_API_KEY")
}

// ViewController
import GoogleMaps

class MapViewController: UIViewController {
    
    override func loadView() {
        let camera = GMSCameraPosition.camera(
            withLatitude: 20.2961,
            longitude: 85.8245,
            zoom: 15
        )
        let mapView = GMSMapView.map(withFrame: .zero, camera: camera)
        view = mapView
        
        let marker = GMSMarker()
        marker.position = CLLocationCoordinate2D(latitude: 20.2961, longitude: 85.8245)
        marker.title = "Office"
        marker.map = mapView
    }
}
```

## Location Services

### Android - FusedLocationProvider

```kotlin
class LocationManager(private val context: Context) {
    
    private val fusedLocationClient = LocationServices.getFusedLocationProviderClient(context)
    
    @SuppressLint("MissingPermission")
    fun getCurrentLocation(onLocation: (Location) -> Unit) {
        fusedLocationClient.lastLocation.addOnSuccessListener { location ->
            location?.let { onLocation(it) }
        }
    }
    
    @SuppressLint("MissingPermission")
    fun startLocationUpdates(onLocation: (Location) -> Unit) {
        val request = LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, 10000)
            .setMinUpdateIntervalMillis(5000)
            .build()
        
        val callback = object : LocationCallback() {
            override fun onLocationResult(result: LocationResult) {
                result.lastLocation?.let { onLocation(it) }
            }
        }
        
        fusedLocationClient.requestLocationUpdates(request, callback, Looper.getMainLooper())
    }
}
```

### iOS - CoreLocation

```swift
import CoreLocation

class LocationManager: NSObject, CLLocationManagerDelegate {
    
    private let manager = CLLocationManager()
    var onLocationUpdate: ((CLLocation) -> Void)?
    
    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
    }
    
    func requestPermission() {
        manager.requestWhenInUseAuthorization()
    }
    
    func startUpdating() {
        manager.startUpdatingLocation()
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        onLocationUpdate?(location)
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("Location error: \(error)")
    }
}
```

### Permissions

```xml
<!-- Android - AndroidManifest.xml -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```

```xml
<!-- iOS - Info.plist -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>We need your location to show nearby places</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>We need background location for navigation</string>
```

## Mapbox

```kotlin
// Android
MapboxMap mapboxMap = mapView.getMapboxMap()
mapboxMap.loadStyleUri(Style.MAPBOX_STREETS) { style ->
    // Add marker
    val point = Point.fromLngLat(85.8245, 20.2961)
    val annotationApi = mapView.annotations
    val pointAnnotationManager = annotationApi.createPointAnnotationManager()
    
    val pointAnnotationOptions = PointAnnotationOptions()
        .withPoint(point)
        .withIconImage(BitmapFactory.decodeResource(resources, R.drawable.marker))
    
    pointAnnotationManager.create(pointAnnotationOptions)
}
```

## ArcGIS Runtime SDK

```kotlin
// Android - Basic Map
class ArcGISMapActivity : AppCompatActivity() {
    
    private lateinit var mapView: MapView
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Set API key
        ArcGISRuntimeEnvironment.setApiKey("YOUR_API_KEY")
        
        mapView = MapView(this)
        setContentView(mapView)
        
        // Create map with basemap
        val map = ArcGISMap(BasemapStyle.ARCGIS_TOPOGRAPHIC)
        mapView.map = map
        
        // Set viewpoint
        mapView.setViewpoint(Viewpoint(20.2961, 85.8245, 50000.0))
    }
    
    // Add feature layer
    fun addFeatureLayer() {
        val featureTable = ServiceFeatureTable("https://services.arcgis.com/.../FeatureServer/0")
        val featureLayer = FeatureLayer(featureTable)
        mapView.map?.operationalLayers?.add(featureLayer)
    }
}
```

### GIS Concepts

```
┌─────────────────────────────────────────┐
│              GIS Layers                 │
├─────────────────────────────────────────┤
│  Points   │ Cities, POIs, markers       │
│  Lines    │ Roads, rivers, boundaries   │
│  Polygons │ Countries, zones, areas     │
│  Raster   │ Satellite imagery, terrain  │
└─────────────────────────────────────────┘
```

## Geofencing

```kotlin
// Android
val geofence = Geofence.Builder()
    .setRequestId("office")
    .setCircularRegion(20.2961, 85.8245, 100f) // lat, lng, radius in meters
    .setExpirationDuration(Geofence.NEVER_EXPIRE)
    .setTransitionTypes(Geofence.GEOFENCE_TRANSITION_ENTER or Geofence.GEOFENCE_TRANSITION_EXIT)
    .build()

val request = GeofencingRequest.Builder()
    .setInitialTrigger(GeofencingRequest.INITIAL_TRIGGER_ENTER)
    .addGeofence(geofence)
    .build()

geofencingClient.addGeofences(request, pendingIntent)
```

## Questions & Answers

> [!question]- Q1: What is the difference between GPS, Network, and Fused location?
> **Answer:**
> - **GPS**: Satellite-based, accurate (5-10m), slow, battery drain
> - **Network**: Cell towers/WiFi, less accurate (100m+), fast, low battery
> - **Fused**: Combines both intelligently, best of both worlds
> 
> Always use FusedLocationProvider on Android.

> [!question]- Q2: How do you handle location permissions in Android 10+?
> **Answer:**
> Android 10+ requires:
> - `ACCESS_FINE_LOCATION` for precise location
> - `ACCESS_BACKGROUND_LOCATION` separately for background access
> - Must request foreground permission first, then background
> 
> Show rationale explaining why background is needed.

> [!question]- Q3: What is a Geofence?
> **Answer:**
> Virtual boundary around a geographic area. Triggers events when device:
> - Enters the area
> - Exits the area
> - Dwells (stays for duration)
> 
> Use cases: Location reminders, attendance, marketing.

> [!question]- Q4: How do you optimize battery when using location?
> **Answer:**
> - Use appropriate accuracy (don't always use HIGH)
> - Increase update interval when possible
> - Stop updates when not needed
> - Use geofencing instead of continuous polling
> - Use significant location changes (iOS)

> [!question]- Q5: What is the difference between Google Maps and Mapbox?
> **Answer:**
> | Google Maps | Mapbox |
> |-------------|--------|
> | Easier setup | More customizable |
> | Better POI data | Custom map styles |
> | Usage-based pricing | More predictable pricing |
> | Limited offline | Better offline support |

> [!question]- Q6: What is ArcGIS used for?
> **Answer:**
> Enterprise GIS platform for:
> - Complex geospatial analysis
> - Custom map layers and data
> - Offline map packages
> - Field data collection
> - Integration with enterprise GIS systems
> 
> Common in government, utilities, logistics.

> [!question]- Q7: How do you show user's current location on map?
> **Answer:**
> ```kotlin
> // Android - Google Maps
> map.isMyLocationEnabled = true
> map.uiSettings.isMyLocationButtonEnabled = true
> ```
> 
> Requires location permission. Shows blue dot with accuracy circle.

> [!question]- Q8: What is reverse geocoding?
> **Answer:**
> Converting coordinates to address:
> ```kotlin
> val geocoder = Geocoder(context)
> val addresses = geocoder.getFromLocation(lat, lng, 1)
> val address = addresses?.firstOrNull()?.getAddressLine(0)
> ```
> 
> Opposite: Geocoding (address → coordinates)

> [!question]- Q9: How do you handle location when app is in background?
> **Answer:**
> - **Android**: Foreground service with notification, or WorkManager for periodic
> - **iOS**: Background location mode, significant location changes
> 
> Both platforms restrict background location for battery. Justify to user.

> [!question]- Q10: What is a Feature Layer in ArcGIS?
> **Answer:**
> Layer that displays geographic features from a feature service:
> - Points, lines, polygons
> - Supports querying and filtering
> - Can be edited (if service allows)
> - Rendered based on attributes
> 
> Example: Show all fire hydrants within 1km.
