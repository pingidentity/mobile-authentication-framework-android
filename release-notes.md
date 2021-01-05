# Release Notes

## v1.0.0 - Jan 5th, 2021

Features:

- The Mobile Authentication Framework sends requests to the PingFederate Authentication API and responds to different states by showing possible end-user actions.
- Currently supported flows are device registration and device authorization.

Known issues and limitations

- **Untrusted accessing device with pushless app as authenticating device** (PingID SDK)<br>
The flow of an untrusted accessing device while a pushless app is used as the authenticating device is not supported with the PingFederate Authentication API.
