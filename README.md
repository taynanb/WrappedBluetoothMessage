# WrappedBluetoothMessage
Android-BluetoothChat sample wrapped to a library for purpose to simplify message exchange between devices over bluetooth. 

Usage
---------------
Declare into your class member object for Bluetooth and Message Services:
```java
private BluetoothMessageService mMessageService;
```

Initialize BluetoothMessageService. The init() method will throw and BluetoothNotAvailableException if there's no Bluetooth present on device.
```java
private void initBluetoothMessageService() {
    FragmentActivity activity = getActivity();
    try {
        mMessageService = BluetoothMessageService.Companion.init(activity);
    } catch (BluetoothNotAvailableException e) {
        e.printStackTrace();
        Toast.makeText(activity, "Bluetooth is not available", Toast.LENGTH_LONG).show();
        activity.finish();
    }
}
```

Use BluetoothMessageService instance to check if bluetooth is enable. If is not, call requestEnableBt() to enable it. The method will call an Android Activity to list and pick available devices.
```java
if (!mMessageService.isBtEnabled()) {
    mMessageService.requestEnableBt();
}
```

For listen to BluetoothMessageService callbacks, implement BluetoothAdapterListener on your class, override its methods and set the listener implementation into BluetoothAdapterManager.
```java
public class BluetoothChatFragment extends Fragment
        implements BluetoothAdapterListener, BluetoothDeviceListener, BluetoothMessageListener { 
        // Implement the listener into your class
        
    private BluetoothMessageService mMessageService;
        
        ...
        
    @Override
    public void onResume() {
        super.onResume();

        // Performing this check in onResume() covers the case in which BT was
        // not enabled during onStart(), so we were paused to enable it...
        // onResume() will be called when ACTION_REQUEST_ENABLE activity returns.
        if (mMessageService != null) {

            mMessageService.setBluetoothAdapterListener(this);
            mMessageService.setBluetoothDeviceListener(this);
            mMessageService.setBluetoothMessageListener(this);

            // Only if the state is STATE_NONE, do we know that we haven't started already
            if (mMessageService.getState() == BluetoothMessageService.STATE_NONE) {
                // Start the Bluetooth chat services
                mMessageService.start();
            }
        }
    }
    
    // Dont forget to remove listener when your screen will be remove
    @Override
    public void onDestroy() {
        super.onDestroy();
        if (mMessageService != null) {

            // stop service
            mMessageService.stop();
            
            // Then remove listeners
            mMessageService.setBluetoothDeviceListener(null);
            mMessageService.setBluetoothMessageListener(null);
            mMessageService.setBluetoothAdapterListener(null);
        }
    }
    
    // Override BluetoothAdapterListener methods
    @Override
    public void onBtEnabled() {
        // User did enable Bluetooth
        setupChat();
    }

    @Override
    public void onBtDisabled() {
        // User did not enable Bluetooth or an error occurred
        Toast.makeText(getActivity(), "BT not enabled",
                Toast.LENGTH_SHORT).show();
        getActivity().finish();
    }
    
    // Override BluetoothDeviceListener methods
    @Override
    public void onDeviceStateConnected(String deviceName) {
        setStatus(getString(R.string.title_connected_to,
                mMessageService.getMConnectedDeviceName()));
        mConversationArrayAdapter.clear();
    }

    @Override
    public void onDeviceStateConnecting() {
        setStatus(R.string.title_connecting);
    }

    @Override
    public void onDeviceStateNotConnected() {
        setStatus(R.string.title_not_connected);
    }

    @Override
    public void onDeviceStateDisconnected() {
        Toast.makeText(getActivity(), R.string.message_device_connection_lost,
                Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onDeviceStateConnectionFailed() {
        Toast.makeText(getActivity(), R.string.message_unable_to_connect_device,
                Toast.LENGTH_SHORT).show();
    }
    
    // Override BluetoothMessageListener methods
    @Override
    public void onMessageWrite(String message) {
        mConversationArrayAdapter.add("Me:  " + message);
    }

    @Override
    public void onMessageRead(String message) {
        mConversationArrayAdapter.add(mMessageService.getMConnectedDeviceName() + ":  " + message);
    }
        
}
```


Use methods to interact with Bluetooth:
```java

// Launch the DeviceListActivity to see devices and do scan.
// true for secure connection or false for insecure connection.
mMessageService.startBtConnection(true);

// Ensure this device is discoverable by others
mMessageService.ensureDiscoverable();
```

Reading connection state:
```java
// Use this method do read connection state
mMessageService.getState()

// Constants that indicate the current connection state
BluetoothMessageService.STATE_NONE = 0       // we're doing nothing
BluetoothMessageService.STATE_LISTEN = 1     // now listening for incoming connections
BluetoothMessageService.STATE_CONNECTING = 2 // now initiating an outgoing connection
BluetoothMessageService.STATE_CONNECTED = 3  // now connected to a remote device

```

After connection stabilished send message to another Bluetooth Device
```java
private void sendMessage(String message) {
    // Check that we're actually connected before trying anything
    if (mMessageService.getState() != BluetoothMessageService.STATE_CONNECTED) {
        Toast.makeText(getActivity(), R.string.not_connected, Toast.LENGTH_SHORT).show();
        return;
    }

    // Check that there's actually something to send
    if (message.length() > 0) {
        mMessageService.write(message);
    }
}
```
