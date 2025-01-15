# Schrodinger-s-Wallpaper
A Wallpaper App that combines creativity with engineering principles to deliver a unique wallpaper UX inspired by quantum mechanics. By making use of two layers of randomness just like the double slit, with true quantum API randomness and optimized for seamless functionality both online and offline.

# Technical Documentation

## Overview

The **Schrödinger's Wallpaper App** is an Android application inspired by the famous Schrödinger's Cat thought experiment in quantum mechanics. The app dynamically changes the wallpaper each time the phone is unlocked, selecting between two categories—*lively cats* and *sleepy cats*. This selection process mimics quantum superposition and wavefunction collapse, creating an engaging and unpredictable user experience.

This document explains the app's concept, algorithm, and implementation in detail, adhering to Microsoft’s guidelines for clarity, consistency, and precision.

---

## Concept

### Schrödinger's Thought Experiment
Schrödinger's Cat is a theoretical experiment where a cat in a sealed box is simultaneously alive and dead until observed. Observation resolves this uncertainty into a single state, representing the collapse of the quantum wavefunction.

### App Inspiration
The app mirrors this principle:
1. Two wallpapers (lively and sleepy cats) remain in a conceptual "superposition" during lock.
2. The wallpaper is only determined upon unlocking, simulating the wavefunction collapse.
3. Randomness introduces unpredictability, with optional integration of quantum-based randomness for authenticity.

---

## Features

1. **Dynamic Wallpaper Updates**:
   - Automatically switches between lively and sleepy cat wallpapers upon unlocking.

2. **Quantum-Enhanced Randomness**:
   - Combines two sources of randomness:
     - **Quantum Randomness API**: Fetches true random numbers based on quantum phenomena when online.
     - **Local Randomness**: Generates pseudorandom numbers when offline or when conditions favor local randomness.

3. **Optimized Performance**:
   - Preloads wallpapers during the lock state for a seamless experience.
   - Limits API calls to reduce latency and optimize resource usage.

4. **Offline Functionality**:
   - Operates entirely offline, relying on local randomness.

---

## Algorithm Design

### Core Principles

1. **Superposition State**:
   - During the lock state, both wallpaper categories coexist in a conceptual superposition.

2. **Wavefunction Collapse**:
   - The wallpaper selection is probabilistic and occurs only when the phone is unlocked.

3. **Dual Randomness Layers**:
   - The first layer determines whether quantum or local randomness will be used.
   - The second layer decides which wallpaper (lively or sleepy) to display.

---

## Workflow

#### Step 1: Lock Event
1. Preload lively and sleepy cat wallpapers into memory.
2. Generate a first-layer random number between 1–100.
3. Defer the network check until the unlock event for accuracy.

#### Step 2: Unlock Event
1. Use Android’s `ConnectivityManager` to determine if the device is online.
2. Evaluate the first-layer random number:
   - If it falls within specific API-triggering values, fetch quantum randomness (if online).
   - Otherwise, or if offline, rely on local randomness.
3. Generate a second-layer random number (0 or 1) to finalize the wallpaper choice:
   - `0`: Lively cat.
   - `1`: Sleepy cat.
4. Set the selected wallpaper instantly.

---

### Decision Logic

#### First Layer: Quantum or Local Randomness
- A random number (1–100) is generated during the lock event.
- Predefined values trigger API-based randomness (e.g., 7, 23, 65).
- All other values fall back to local randomness.

#### Second Layer: Wallpaper Selection
- The result of the first layer determines which randomness source generates the second-layer value:
   - `0`: Apply the lively cat wallpaper.
   - `1`: Apply the sleepy cat wallpaper.

---

## Implementation Details

### Wallpaper Preloading
Both wallpapers are preloaded during the lock state using Android's `BitmapFactory` for smooth transitions:
```kotlin
val livelyCatBitmap = BitmapFactory.decodeResource(context.resources, R.drawable.lively_cat)
val sleepyCatBitmap = BitmapFactory.decodeResource(context.resources, R.drawable.sleepy_cat)
```

### Quantum Randomness Integration
Quantum randomness is fetched via APIs like [ANU QRNG](https://qrng.anu.edu.au):
```kotlin
fun fetchQuantumRandomNumber(): Int? {
    return try {
        val response = URL("https://qrng.anu.edu.au/API/jsonI.php?length=1&type=uint8").readText()
        val jsonObject = JSONObject(response)
        jsonObject.getJSONArray("data").getInt(0) % 2
    } catch (e: Exception) {
        null
    }
}
```

### Local Randomness Fallback
Secure pseudorandom numbers are generated locally when offline:
```kotlin
fun generateFallbackRandom(): Int {
    val secureRandom = SecureRandom()
    return secureRandom.nextInt(2)
}
```

### Network Status Check
The device’s online status is checked at unlock using `ConnectivityManager`:
```kotlin
fun isDeviceOnline(): Boolean {
    val connectivityManager = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
    return connectivityManager.activeNetworkInfo?.isConnected == true
}
```

---

## Code Example

Below is a simplified implementation of the core logic:

```kotlin
class SchrodingersWallpaperApp {

    private var livelyCatBitmap: Bitmap? = null
    private var sleepyCatBitmap: Bitmap? = null
    private var firstLayerRandom: Int = -1

    fun onPhoneLocked() {
        livelyCatBitmap = BitmapFactory.decodeResource(context.resources, R.drawable.lively_cat)
        sleepyCatBitmap = BitmapFactory.decodeResource(context.resources, R.drawable.sleepy_cat)
        firstLayerRandom = (1..100).random()
    }

    fun onPhoneUnlocked() {
        val isOnline = isDeviceOnline()
        val finalRandom = if (shouldUseQuantumAPI(firstLayerRandom) && isOnline) {
            fetchQuantumRandomNumber() ?: generateFallbackRandom()
        } else {
            generateFallbackRandom()
        }
        setWallpaper(if (finalRandom == 0) livelyCatBitmap else sleepyCatBitmap)
    }

    private fun shouldUseQuantumAPI(randomNumber: Int): Boolean {
        return randomNumber in listOf(7, 23, 65)
    }

    private fun setWallpaper(bitmap: Bitmap?) {
        WallpaperManager.getInstance(context).setBitmap(bitmap)
    }
}
```

---

## Testing and Validation

| Scenario                              | Expected Behavior                                   |
|---------------------------------------|---------------------------------------------------|
| Device offline                        | Wallpaper determined by local randomness.         |
| Device online, API-triggering random  | Wallpaper determined by quantum randomness.       |
| Device online, non-triggering random  | Wallpaper determined by local randomness.         |
| Network lost after lock               | Local randomness used without errors.             |

---

## Conclusion

The **Schrödinger's Wallpaper App** combines creative conceptualization with robust engineering. By integrating two layers of randomness and optimizing for performance, the app delivers a unique and seamless experience inspired by quantum mechanics. It operates reliably both online and offline, embodying innovation and efficiency.
