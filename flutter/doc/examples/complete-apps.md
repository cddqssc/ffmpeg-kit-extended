# Complete Application Architectures

This page outlines the architectural patterns for building full-scale media applications using FFmpeg Kit Extended.

## 1. Video Converter App

A standard converter app needs to handle file selection, parameter configuration, and background processing.

### Converter Architecture

- **State Management**: Use `Provider` or `Bloc` to track the list of tasks.
- **Service Layer**: Create an `FFmpegService` that wraps `FFmpegKit.executeAsync`.
- **UI Components**:
  - `TaskCard`: Shows progress, speed, and status for an individual conversion.
  - `SettingsPanel`: Allows user to pick resolution, bitrate, and codec.

### Key Snippet: Task Management

```dart
class ConversionTask {
  final String input;
  final String output;
  double progress = 0;
  SessionState state = SessionState.created;
  
  void run() {
    FFmpegKit.executeAsync(
      '-i $input $output',
      onStatistics: (s) => progress = calculateProgress(s),
      onComplete: (s) => state = s.getState(),
    );
  }
}
```

## 2. Media Player with Metadata

A media player that displays rich metadata and supports seeking.

### Player Architecture

- **FFprobe Integration**: Load metadata when the file is selected.
- **FFplay UI**: Use `FFplayKit` for the playback window and build a custom overlay for controls.
- **Background Audio**: Use `audio_service` plugin if you want the audio to continue when the app is minimized (FFmpeg Kit processes, but audio session management is standard Flutter).

### Key Snippet: Metadata Refresh

```dart
void onLoadMedia(String path) async {
  final session = await FFprobeKit.getMediaInformationAsync(path);
  final info = session.getMediaInformation();
  
  setState(() {
    currentVideoTitle = info?.tags?['title'] ?? 'Unknown';
    duration = parseDuration(info?.duration);
  });
  
  FFplayKit.execute(path);
}
```

## 3. Video Editing Suite

For apps like "Trimmer" or "Filter App" where the user sees a preview.

### Editor Architecture

- **Preview Engine**: Use `FFplayKit` to show a preview of the video at a specific timestamp.
- **Live Preview**: When the user adjusts a slider (e.g., brightness), you can run a quick `FFmpegKit` command to generate a thumbnail of that frame with the filter applied.
- **Exporting**: Queue the final high-quality export using `executeAsync`.

### Key Snippet: Live Filter Preview

```dart
void onBrightnessChanged(double value) {
  // Generate a thumbnail frame with current filter
  FFmpegKit.execute(
    '-ss $currentTime -i $input -vf "eq=brightness=$value" -vframes 1 preview.jpg'
  );
  // Update UI with preview.jpg
}
```

## Design Considerations

1. **Power Management**: Media processing is CPU intensive. Use `wakelock` plugin to prevent the device from sleeping during long conversions.
2. **Notification Integration**: For long-running background tasks, use `flutter_local_notifications` to show progress in the system tray.
3. **Storage Access**: On Android 11+, ensure you are using Scope Storage correctly or have the `MANAGE_EXTERNAL_STORAGE` permission for arbitrary file access.
