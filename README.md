# RemoteUi

Endpoint-defined menus for ESP-NOW remotes (e.g. the dual 4D joystick remote).

The endpoint (robot, pick-and-place machine, ...) owns the menu and the icons.
On connect the remote downloads a compact description of the menu, renders it
locally and sends value changes and actions back. Navigation runs on the remote
without radio round trips. Independently, the remote streams its raw input state
(both sticks: x, y, z, button) at a fixed rate; each endpoint decides what to do
with it.

Status: planning. The wire protocol is not defined yet.

## Wire rules (fixed)

ESP-NOW frames are shared with ESPNowManager and with legacy endpoints that use
`remoteProto.h` (`[deviceType, cmd, payload...]`). RemoteUi frames must not be
mistaken for either:

- **Byte 0** must not collide with ESPNowManager message types
  (`0x01..0x04`, `0x10`, `0x20/0x21`, `0x30/0x31`, `0x40/0x41`).
  ESPNowManager dispatches on byte 0 before user data reaches the application.
- **Byte 1** must not be a legacy `ctrlCmd` value (`0..7`). Legacy robot
  controllers ignore byte 0 and execute byte 1 as a motion command.
  RemoteUi message types therefore live in the range `0xC0..0xFF`.

## Planned parts

- Protocol structs (packed, little-endian, versioned)
- Server (endpoint side): serializes a `Menu` tree, serves menu and icon blobs
  on request, input stream watchdog (stop motion when the stream stops)
- Client (remote side): downloads and caches menus/icons (LittleFS), builds
  local menu objects whose actions send frames back
