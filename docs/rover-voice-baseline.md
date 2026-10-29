# Rover voice-board baseline

This checkout is the voice side of the FNK0053 modular rover project. The Freenove ESP32 remains responsible for movement; this ESP32-S3 handles audio. No rover-control integration is included in this baseline.

## Source and hardware

- Fork: https://github.com/kagedws6/xiaozhi-esp32
- Upstream: https://github.com/78/xiaozhi-esp32
- Baseline source: `c435f99a88066029ec2dbd2d758c8cc9d051565b` (project version 2.5.3).
- Board directory: `waveshare/esp32-s3-audio-board`.
- Firmware variant: `esp32-s3-audio-board`.
- ESP32-S3 with 16 MB flash and 8 MB PSRAM, confirmed over USB on 2026-10-27.
- Original lower housing and speaker removed; owner has a separate speaker. Housing changes do not change the electronic board identity.

## Reproduce the baseline

Activate a supported ESP-IDF environment first. This baseline attempt uses the existing ESP-IDF v6.0.2 installation. Upstream recommends release/v6.1; its minimum is v6.0.1. Use the canonical builder rather than editing generated sdkconfig files:

```sh
python scripts/build.py waveshare/esp32-s3-audio-board --name esp32-s3-audio-board --language en-US --wake-word hiesp --zip
```

English and the standard "Hi ESP" wake word are explicit build options. Board pins and application code are unchanged. The stock board implementation initializes its optional display and camera interfaces; physical operation without those accessories must be checked on this assembly. The rover camera is on the separate Freenove controller.

On this Windows checkout, the first build hit Ninja's command-length limit near linking. Apply upstream's documented response-file workaround, then repeat the canonical builder command above:

```sh
idf.py -D CMAKE_NINJA_FORCE_RESPONSE_FILE=ON reconfigure
```

This is a CMake cache option, not a firmware-source modification. Repeat it after clearing the build cache.

The subsequent link failed because two vendor audio/image library paths were split at the space in `New Ideas`. The active Windows build checkout is therefore `D:\esp32-rover\voice-baseline`, at the same baseline source commit. Its `origin` is the user's fork and `upstream` is the original project. The original workspace checkout remains available, but its build directory contains failed attempts. For a fresh short-path build, setting `CMAKE_NINJA_FORCE_RESPONSE_FILE=1` in the process environment enables response files from the first configuration.

The original 16 MB firmware backup completed on 2026-10-27. Verification against the board with `esptool verify-flash` succeeded over the full flash capacity. The backup, its checksum manifest and restore instructions are private local files, not repository content. Several regions required the ROM reader after the flasher stub stopped transferring data; all chunks were assembled and the complete backup was verified against the device.

Expected outputs are `build/merged-binary.bin`, `build/flasher_args.json` and a ZIP in `releases/`. A build is successful only when the builder finishes and the files and partition layout are checked.

## Backup before flash

Keep the existing firmware and provisioning data in a private local backup outside every public repository. Do not publish raw flash dumps, NVS contents, serial logs containing credentials, or device identity data. Record the backup's full size and SHA-256. A backup read must complete before replacing flash. Read failures must not be mistaken for complete backups.

Use the connected device's detected flash capacity, and verify the backup against a second read before flashing. Do not burn eFuses, change security settings, or force flashing past a security error. This board reports secure boot and flash encryption disabled.

## Hardware smoke test

After a verified flash, check boot stability, speaker output at a modest volume, microphone capture, wake-word detection, Wi-Fi provisioning, and a voice exchange. Wi-Fi credentials should be entered locally by the owner. Complete any required voice-service activation locally as well. Confirm that reconnecting works. Record each result individually; a successful upload does not verify microphone or speaker operation.

The baseline is not a fully offline assistant: local wake-word detection and the conversational backend are separate. Service setup and network access are needed for the selected conversational service. No movement commands should be added until the rover implements command expiry and validates requests independently.

## Results on 2026-10-27

- Short-path baseline build and ZIP packaging succeeded with ESP-IDF v6.0.2 and response files enabled.
- Application size: 2,648,144 bytes; OTA app partition capacity: 4,128,768 bytes. Image checksum and validation hash passed.
- Flashing the merged image after clearing old settings succeeded; written-data hash verification passed.
- A 40-second boot capture showed one boot, no panic/abort, successful ES8311/ES7210 audio-codec initialization, and entry into Wi-Fi provisioning mode.
- Logs show successful audio output initialization and setup-prompt playback. The owner confirmed the setup speech was audible through the replacement speaker; speech during the subsequent restart was not separately confirmed.
- The stock board implementation probes for OV2640/OV5640 cameras. Those probes failed on the voice board without a camera; initialization continued into setup mode. No camera support has been removed or modified. The Freenove rover camera remains independent.
- Home Wi-Fi provisioning succeeded. A subsequent restart reconnected, obtained a DHCP address, completed service activation, and reached the idle state without a panic in the 40-second observation.
- The audio front end and `wn9_hiesp` wake-word model initialized, and the firmware announced its "Hi ESP" wake-word hint. The owner confirmed that saying "Hi ESP" and asking "What can you do?" woke the assistant and produced an audible answer. This verifies the basic microphone/wake-word/conversational-service/speaker path for this interaction; range, motor-noise performance and extended reliability remain untested.

See `rover-voice-baseline-validation.json` for build provenance. This source commit is a development baseline, not a claim that it matches the board's original factory image. Firmware application code and board pins are unchanged.
