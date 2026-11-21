# How should Clips transfer to an Android phone?

Resolves research ticket #5. Terms follow `CONTEXT.md` (Pendant, Clip, Transfer, Preview, Onboard Storage).
Sources were read on 2026-11-21 and cover Android docs through Android 16 (API 36). Android 17 was not checked.
Items marked **[EST]** are my estimates, not sourced facts. Items marked **[UNVERIFIED]** need a bench test.

## Recommendation

1. **BLE for the control plane.** Pair once through CompanionDeviceManager (CDM). BLE carries wake, status, the Wi-Fi credentials and the Transfer manifest. It does not carry Clips.
2. **Pendant-hosted Wi-Fi AP as the bulk and Preview link.** The phone joins with `WifiNetworkSpecifier`, pinned to the SSID and BSSID. After the first approval, later requests for that same network are silent. Clips move over TCP/HTTP with per-Clip resume.
3. **Fallback: phone-hosted `LocalOnlyHotspot`**, with the Pendant joining as a Wi-Fi client. Use it if the pendant chip cannot run a stable SoftAP, or if the specifier path fails on a target phone.
4. **Wi-Fi Direct and Wi-Fi Aware are not first choice.** They need P2P or NAN firmware on the Pendant, and I found no source showing it interoperates with Android.
5. **BLE-only is not viable for Clips.** It is roughly 6 to 23 minutes per 1-minute clip, and cannot carry a Preview.

Why the pendant-hosted AP wins:
- Its credentials are fixed at pairing, so there is no per-session secret exchange.
- The pendant's SoftAP stays off except during Transfer, so the cost lands on the Pendant only while it is moving data.
- The phone does not have to run a hotspot.
- A pinned request can skip the user prompt after the first approval ([bootstrap doc](https://developer.android.com/develop/connectivity/wifi/wifi-bootstrap)).

## Transfer-time estimate (1-minute 1080p Clip)

Clip size is 60 MB at 8 Mbps and 120 MB at 16 Mbps (decimal; 480 and 960 Mbit).

| Link | Assumed sustained rate | Basis | 8 Mbps Clip | 16 Mbps Clip |
|---|---|---|---|---|
| BLE GATT, 1M PHY | ~0.7 Mbps | Nordic nRF52 SoftDevice table: 703 to 771 kbps with a 247-byte MTU ([Nordic](https://docs.nordicsemi.com/r/bundle/sds_s140/page/sds/s1xx/ble_data_throughput/ble_data_throughput.html)) | ~11 min **[EST]** | ~23 min **[EST]** |
| BLE GATT, 2M PHY | ~1.3 Mbps | Same table: 1327 to 1376 kbps | ~6 min **[EST]** | ~12 min **[EST]** |
| Wi-Fi (SoftAP, LOH, Direct) | 20 to 50 Mbps | 20 Mbps is the ESP32-S3 over-the-air TCP figure; 50 Mbps is my guess for a good 5 GHz link ([Espressif](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/wifi-driver/wifi-performance-and-power-save.html)) | 10 to 24 s **[EST]** | 19 to 48 s **[EST]** |

- Nordic's figures assume an interference-free link, an encrypted connection, and an app that never stalls the pipe. Real phones will be lower **[EST]**.
- Add about 5 to 15 s of Wi-Fi bring-up per Transfer session **[EST]**.
- SD-card read speed and flash write speed on either end may cap the Wi-Fi case **[UNVERIFIED]**.
- Chip choice for the Pendant is not yet made. The Wi-Fi numbers are a proxy from one candidate MCU.
- A 16 Mbps Clip transfers faster than real time only if the link sustains more than 16 Mbps. That is plausible on Wi-Fi and impossible on BLE.

**Preview:** a 480p to 720p stream at about 2 to 4 Mbps **[EST]** fits Wi-Fi comfortably. It exceeds any BLE rate above, so Preview requires the Wi-Fi link.

## Comparison

| | BLE + Wi-Fi Direct | Phone `LocalOnlyHotspot` | Pendant AP + `WifiNetworkSpecifier` | BLE only | Wi-Fi Aware (extra) |
|---|---|---|---|---|---|
| Real throughput | Wi-Fi class, 20+ Mbps **[EST]** | Wi-Fi class **[EST]** | Wi-Fi class; band request possible (`getBand` API 31, preferred channels API 34) ([ref](https://developer.android.com/reference/android/net/wifi/WifiNetworkSpecifier)) | 0.7 to 1.4 Mbps | Documented as higher than Bluetooth; no figure ([doc](https://developer.android.com/develop/connectivity/wifi/wifi-aware)) |
| Preview | Yes | Yes | Yes | No | Yes, if supported |
| Pendant firmware | P2P stack needed; embedded support **[UNVERIFIED]** | Wi-Fi client only (simplest) | SoftAP (common) | GATT only | NAN needed; support **[UNVERIFIED]** |
| Pendant power | Radio on during group; unmeasured | Client radio only; likely lowest Pendant cost **[EST]** | AP beaconing during Transfer; likely highest Pendant cost **[EST]** | Lowest ([BLE overview](https://developer.android.com/develop/connectivity/bluetooth/ble/ble-overview)) | Clustering uses battery ([doc](https://developer.android.com/develop/connectivity/wifi/wifi-aware)) |
| Phone power | Medium **[EST]** | Higher: phone runs an AP **[EST]** | Medium **[EST]** | Lowest | Medium **[EST]** |
| Phone permissions | `NEARBY_WIFI_DEVICES` (API 33+), or `ACCESS_FINE_LOCATION` (to API 32); Location mode must be on for discovery ([doc](https://developer.android.com/develop/connectivity/wifi/wifi-direct)) | `NEARBY_WIFI_DEVICES` (target 33+), else `ACCESS_FINE_LOCATION` ([doc](https://developer.android.com/develop/connectivity/wifi/localonlyhotspot)) | Wi-Fi state permissions; user dialog on first request ([doc](https://developer.android.com/develop/connectivity/wifi/wifi-bootstrap)) | `BLUETOOTH_SCAN`/`CONNECT`; CDM avoids location ([doc](https://developer.android.com/develop/connectivity/bluetooth/bt-permissions)) | `NEARBY_WIFI_DEVICES`; `FEATURE_WIFI_AWARE` check ([doc](https://developer.android.com/develop/connectivity/wifi/wifi-aware)) |
| User prompts | Pendant must auto-accept; phone-side prompt behavior not in the doc **[UNVERIFIED]** | None documented | First connect only; a specific SSID/BSSID request is then auto-approved; a pattern request always prompts | Only CDM association | Data-path approval behavior not in the doc **[UNVERIFIED]** |
| Phone keeps internet | Not documented **[UNVERIFIED]** | The hotspot network has no internet ([doc](https://developer.android.com/develop/connectivity/wifi/localonlyhotspot)); phone's own Wi-Fi survives only on STA+AP concurrent hardware ([AOSP](https://source.android.com/docs/core/connect/wifi-sta-ap-concurrency)) | The link has no internet ([doc](https://developer.android.com/develop/connectivity/wifi/wifi-bootstrap)); primary Wi-Fi can stay up as a secondary-connection case only when `isStaConcurrencyForLocalOnlyConnectionsSupported()` is true and the app targets API 31+ ([ref](https://developer.android.com/reference/android/net/wifi/WifiNetworkSpecifier)). Otherwise assume Wi-Fi internet drops and mobile data takes over **[EST]** | Yes | Aware data path is a separate network; coexistence varies by device ([doc](https://developer.android.com/develop/connectivity/wifi/wifi-aware)) |
| Reconnect | Re-run discovery and connect | Credentials are new each reservation (see risks); user can stop it at any time ([ref](https://developer.android.com/reference/android/net/wifi/WifiManager.LocalOnlyHotspotCallback)) | Re-issue `requestNetwork` per session; user "forget" clears the stored approval | CDM presence plus GATT reconnect | Availability can change at any time; discard and re-create sessions |

## Background execution and companion pairing

- **CDM association** gives a system pairing dialog, scans without location permission, and can mean no `ACCESS_FINE_LOCATION` request. It does not itself create connections ([doc](https://developer.android.com/develop/connectivity/bluetooth/companion-device-pairing)).
- **Presence-triggered start.** On Android 16 (API 36), `startObservingDevicePresence(ObservingDevicePresenceRequest)` binds the app's `CompanionDeviceService` while the Pendant is in BLE range or connected, and unbinds it when it leaves. The older overload and `onDeviceAppeared()` are deprecated ([doc](https://developer.android.com/develop/connectivity/bluetooth/companion-device-pairing)).
- **Starting the foreground service from the background.** An app that uses CDM and declares `REQUEST_COMPANION_START_FOREGROUND_SERVICES_FROM_BACKGROUND` (preferred) or `REQUEST_COMPANION_RUN_IN_BACKGROUND` is exempt from the Android 12+ background-start ban ([doc](https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start)).
- **Service type.** Use `connectedDevice`. Its prerequisites are met by `CHANGE_WIFI_STATE` in the manifest or a granted `BLUETOOTH_*` permission, and Android's own guidance names it for syncing with a connected device ([types](https://developer.android.com/develop/background-work/services/fgs/service-types), [data-transfer options](https://developer.android.com/develop/background-work/background-tasks/data-transfer-options)).
- **Avoid `dataSync`.** For apps targeting API 35+, `dataSync` is capped at 6 hours per 24 hours and calls `onTimeout()`, after which the app must stop ([Android 15 changes](https://developer.android.com/about/versions/15/behavior-changes-15)).
- **`shortService` is too short.** It allows about 3 minutes, and the 120 MB Clip case plus setup can exceed that ([types](https://developer.android.com/develop/background-work/services/fgs/service-types)).
- **Preview** only runs while the user is aiming, with the app in the foreground, so foreground-service rules do not constrain it.
- **User-visible cost.** A foreground service needs a persistent notification during Transfer **[EST: expected; not separately verified]**.

## Proposed connection flow

**One-time pairing**
1. The app asks for Nearby devices permission and starts a CDM `AssociationRequest` filtered on the Pendant's BLE service UUID. The user confirms once.
2. Over the bonded BLE link, the Pendant sends a Wi-Fi SSID, BSSID and random passphrase (or the app provisions them). The app stores them.
3. The app registers for device presence.

**Automatic Transfer**
1. The Pendant finishes a Capture and advertises "Clips pending".
2. The OS sees the Pendant in BLE range and binds the `CompanionDeviceService`, which starts a `connectedDevice` foreground service.
3. The service reads the manifest over GATT (Clip IDs, sizes, hashes) and sends "start AP".
4. The Pendant brings up its AP.
5. The app calls `requestNetwork` with a `WifiNetworkSpecifier` pinned to the stored SSID and BSSID. The first run shows the system approval dialog; later runs skip it.
6. The app opens a socket bound to the returned `Network` and pulls Clips with HTTP range requests. It verifies hashes, acknowledges over GATT or HTTP, and the Pendant marks the Clip safe to delete.
7. The app releases the network callback; the Pendant drops the AP and returns to BLE-only.

**Preview**
1. The user opens the viewfinder and the app sends "start AP + stream" over BLE.
2. The app joins as above and plays the stream.
3. On leaving the screen, the app tears the link down.

**Failure handling**
- If `onUnavailable()` fires, retry once, then drop to the fallback path (below) and tell the user in-app.
- If the link drops mid-Clip, resume by byte offset on the next session.

**Fallback:** phone-hosted `LocalOnlyHotspot`. The phone requests it, reads the generated SSID and passphrase, and sends them over BLE. The Pendant joins as a client.

## Open risks

1. **Phone internet during Transfer.** Whether primary Wi-Fi survives depends on per-device STA concurrency for local-only connections. On other phones the user may fall back to mobile data for the duration. Needs testing on a device matrix.
2. **Pendant SoftAP stability and power.** I found no primary source for SoftAP throughput or current draw on any candidate chip. Espressif's page gives no softAP figures and no mA figures. Measure early.
3. **Silent approval after the first connect.** The bootstrap doc says a specific request skips the dialog, but OEM behavior, Android-version changes, and stored-approval loss after "forget" are untested.
4. **Background reliability.** The CDM presence and foreground-service exemption chain is documented; OEM battery managers and Doze behavior are not covered by these sources. Test on at least three OEMs.
5. **Credentials.** The `LocalOnlyHotspot` doc I read does not say whether the passphrase is random or caller-supplied in all configurations, nor give a user-prompt behavior. Treat it as per-reservation and verify. Also, BLE bonded-link security must protect the Wi-Fi passphrase exchange.
6. **Wi-Fi Direct and Wi-Fi Aware.** Embedded-chip support and phone-side prompts are unverified. Revisit only if SoftAP proves unworkable.
7. **Estimates.** Every throughput and time figure for Wi-Fi is derived from one MCU's benchmark; BLE figures come from a Nordic SoC, not a measured Android phone link.
8. **Privacy.** The BLE overview warns that data over a paired BLE link is visible to all apps on the phone ([doc](https://developer.android.com/develop/connectivity/bluetooth/ble/ble-overview)). Encrypt Clip data and credentials at the application layer.
9. **Android 17.** Not reviewed; re-check foreground-service and CDM changes before release.

## Sources

Android developer docs
- Companion device pairing: https://developer.android.com/develop/connectivity/bluetooth/companion-device-pairing
- Bluetooth permissions: https://developer.android.com/develop/connectivity/bluetooth/bt-permissions
- BLE overview: https://developer.android.com/develop/connectivity/bluetooth/ble/ble-overview
- Wi-Fi Direct: https://developer.android.com/develop/connectivity/wifi/wifi-direct
- Local-only hotspot: https://developer.android.com/develop/connectivity/wifi/localonlyhotspot
- `LocalOnlyHotspotCallback`: https://developer.android.com/reference/android/net/wifi/WifiManager.LocalOnlyHotspotCallback
- Peer-to-peer Wi-Fi (`WifiNetworkSpecifier`): https://developer.android.com/develop/connectivity/wifi/wifi-bootstrap
- `WifiNetworkSpecifier` reference: https://developer.android.com/reference/android/net/wifi/WifiNetworkSpecifier
- Wi-Fi suggestions (contrast; gives internet): https://developer.android.com/develop/connectivity/wifi/wifi-suggest
- Wi-Fi Aware: https://developer.android.com/develop/connectivity/wifi/wifi-aware
- Foreground-service types: https://developer.android.com/develop/background-work/services/fgs/service-types
- Background-start restrictions and exemptions: https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start
- Data-transfer options: https://developer.android.com/develop/background-work/background-tasks/data-transfer-options
- Android 15 behavior changes (`dataSync` timeout): https://developer.android.com/about/versions/15/behavior-changes-15

Platform and vendor
- AOSP Wi-Fi STA/AP concurrency: https://source.android.com/docs/core/connect/wifi-sta-ap-concurrency
- Nordic S140 BLE throughput: https://docs.nordicsemi.com/r/bundle/sds_s140/page/sds/s1xx/ble_data_throughput/ble_data_throughput.html
- Espressif Wi-Fi performance and power save (ESP32-S3): https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/wifi-driver/wifi-performance-and-power-save.html
- Wi-Fi Alliance, Wi-Fi Direct overview (no data rates given): https://www.wi-fi.org/discover-wi-fi/wi-fi-direct
