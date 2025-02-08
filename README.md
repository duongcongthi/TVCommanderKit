# TVCommanderKit

**TVCommanderKit** is a Swift SDK for controlling Samsung Smart TVs over a WebSocket connection. It provides a simple and convenient way to interact with your TV and send remote control commands from your iOS application. This README.md file will guide you through using the TVCommanderKit SDK in your project.

## Table of Contents
- [Usage](#usage)
- [Delegate Methods](#delegate-methods)
- [Searching for TVs](#searching-for-tvs)
- [Fetching TV Device Info](#fetching-tv-device-info)
- [Establishing Connection](#establishing-connection)
- [Authorizing Application with TV](#authorizing-application-with-tv)
- [Sending Remote Control Commands](#sending-remote-control-commands)
- [Disconnecting from TV](#disconnecting-from-tv)
- [Error Handling](#error-handling)
- [Text Entry](#text-entry)
- [Wake on LAN](#wake-on-lan)
- [Managing TV Apps](#managing-tv-apps)
- [License](#license)

## Usage

1. Import the TVCommanderKit module:

```swift
import TVCommanderKit
```

2. Initialize a `TVCommander` object with your TV's IP address and application name:

```swift
let tvCommander = try TVCommander(tvIPAddress: "your_tv_ip_address", appName: "your_app_name")
```

3. Implement the necessary delegate methods to receive updates and handle events (see [Delegate Methods](#delegate-methods)), then set your delegate to receive updates and events from the TVCommander:

```swift
tvCommander.delegate = self
```

4. Connect to your TV (see [Establishing Connection](#establishing-connection) for further details):

```swift
tvCommander.connectToTV()
```

5. Handle incoming authorization steps to authorize your app to send remote controls (see [Authorizing Application with TV](#authorizing-application-with-tv)).

6. Send remote control commands (see [Sending Remote Control Commands](#sending-remote-control-commands)).

7. Disconnect from your TV when done:

```swift
tvCommander.disconnectFromTV()
```

## Delegate Methods

The TVCommanderDelegate protocol provides methods to receive updates and events from the TVCommanderKit SDK:

- `tvCommanderDidConnect(_ tvCommander: TVCommander)`: Called when the TVCommander successfully connects to the TV.

- `tvCommanderDidDisconnect(_ tvCommander: TVCommander)`: Called when the TVCommander disconnects from the TV.

- `tvCommander(_ tvCommander: TVCommander, didUpdateAuthState authStatus: TVAuthStatus)`: Called when the authorization status is updated.

- `tvCommander(_ tvCommander: TVCommander, didWriteRemoteCommand command: TVRemoteCommand)`: Called when a remote control command is sent to the TV.

- `tvCommander(_ tvCommander: TVCommander, didEncounterError error: TVCommanderError)`: Called when the TVCommander encounters an error.

## Searching for TVs

The TVCommanderKit SDK includes a `TVSearcher` class that helps you discover Samsung Smart TVs on the network. You can use this class to search for TVs and obtain `TV` instances for each discovered TV.

```swift
let tvSearcher = TVSearcher()
tvSearcher.addSearchObserver(self)
tvSearcher.startSearch()
```

You can also specify a TV to search for using its ID. Once the `TVSeacher` finds a TV with a matching ID, it will automatically stop searching:

```swift
tvSearcher.configureTargetTVId("your_tv_id")
```

To stop searching for TVs, call the stopFindingTVs() method:

```swift
tvSearcher.stopSearch()
```

You need to conform to the `TVSearchObserving` protocol to receive updates on the search state and discovered TVs.

## Fetching TV Device Info

Upon finding `TV`s using the `TVSearcher` class, you can also fetch device info using the `TVFetcher`.

```swift
let tvFetcher = TVFetcher(session: .shared) // use custom URLSession for mocking, etc
``` 

Invoking the `fetchDevice(for:)` method will return either an updated `TV` object (with device info injected) or a `TVFetcherError`.

```swift
tvFetcher.fetchDevice(for: tv) { result in
    switch result {
    case .success(let tvFetched):
        // use updated tv
    case .failure(let error):
        // handle fetch error
    }
}
```

## Establishing Connection

To establish a connection to your TV, use the `connectToTV()` method. This method will create a WebSocket connection to your TV with the provided IP address and application name. If a connection is already established, it will handle the error gracefully.

```swift
tvCommander.connectToTV()
```

If you're worried about man-in-the-middle attacks, it's recommended that you implement a custom cert-pinning type and pass it through the optional `certPinner` parameter to ensure connection to a trusted server (make sure your custom cert-pinning type doesn't also strongly reference the `TVCommander`, as this will result in a retain cycle).

## Authorizing Application with TV

After establishing a connection to the TV, you'll need to authorize your app for sending remote controls. You can access the authorization status of your application via the `authStatus` property. If your TV hasn't already handled your application's authorization, a prompt should appear on your TV for you to choose an authorization option. Once you select an option within the prompt, if the authorization status updated, it will get passed back through the corresponding delegate method:

```swift
func tvCommander(_ tvCommander: TVCommander, didUpdateAuthState authStatus: TVAuthStatus) {
    switch authStatus {
    case .none: // application authorization incomplete
    case .allowed: // application allowed to send commands, token stored in tvCommander.tvConfig.token
    case .denied: // application not allowed to send commands
    }
}
```

## Sending Remote Control Commands

You can send remote control commands to your TV using the `sendRemoteCommand(key:)` method. Ensure that you are connected to the TV and the authorization status is allowed before sending commands.

```swift
tvCommander.sendRemoteCommand(key: .enter)
```

## Text Entry

**TVCommanderKit** provides a convenient text entry feature, allowing you to quickly input text into on-screen keyboards. There are now two ways to send text input to the TV:

1. **Direct Text Input**  
    Use `sendText(_:)` to send text directly to the TV. This is the preferred method for compatible devices.  
    ```swift
    let textToSend = "Hello, World!"
    tvCommander.sendText(textToSend)
    ```
2. **Keyboard Navigation (Legacy)**  
    The `enterText(_:on:)` method simulates typing by navigating an on-screen keyboard. This approach will likely be removed in future versions.
    ```swift
    let textToEnter = "Hello, World!"
    let keyboardLayout = TVKeyboardLayout.youtube
    tvCommander.enterText(textToEnter, on: keyboardLayout)
    ```

## Disconnecting from TV

When you're done with the TVCommander, disconnect from your TV using the `disconnectFromTV()` method:

```swift
tvCommander.disconnectFromTV()
```

## Error Handling

The TVCommanderKit SDK includes error handling for various scenarios, and errors are reported through the delegate method `tvCommander(_ tvCommander: TVCommander, didEncounterError error: TVCommanderError)`. You should implement this method to handle errors appropriately.

## Wake on LAN

The TVCommanderKit SDK now supports Wake on LAN functionality. You can wake up your TV using the `wakeOnLAN` method:

```swift
let device = TVWakeOnLANDevice(mac: "your_tv_mac_address")
TVCommander.wakeOnLAN(device: device) { error in
    if let error = error {
        print("Wake on LAN failed: \(error.localizedDescription)")
    } else {
        print("Wake on LAN successful!")
    }
}
```

## Managing TV Apps

`TVAppManager` allows you to interact with TV applications on Samsung TVs using their IP address and app ID.

#### Usage

1. **Initialization:**

   ```swift
   let tvAppManager = TVAppManager()
   let tvApp = TVApp(id: "3201907018807", name: "Netflix")
   ```

2. **Fetch App Status:**

   ```swift
   let tvIPAddress = "192.168.1.100"
   let appStatus = try await tvAppManager.fetchStatus(for: tvApp, tvIPAddress: tvIPAddress)
   print(appStatus)
   ```

3. **Launch App:**

   ```swift
   try await tvAppManager.launch(tvApp: tvApp, tvIPAddress: tvIPAddress)
   ```

Errors like invalid URLs or app not found are handled by `TVAppManagerError` and `TVAppNetworkError`.

## License

The TVCommanderKit SDK is distributed under the MIT license.

---

Feel free to contribute to this SDK, report issues, or provide feedback. I hope you find TVCommanderKit useful in building your Samsung Smart TV applications!
