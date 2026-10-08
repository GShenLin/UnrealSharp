# Non-blocking hot reload failures

When a save introduces a compile error (including temporarily incomplete edits across files), the editor reports the failure in Output Log and a notification. It keeps the currently loaded assemblies. No modal dialog needs to be acknowledged before further saves can be processed.

The notification links to Output Log and can be dismissed. Further failures update the existing notification. A successful reload changes it to a success message and expires it.

After a failure, automatic reload waits for another successfully applied source edit. This prevents focus-based reload and exiting PIE from repeatedly compiling unchanged invalid source. Explicit reload remains available for a deliberate retry.

## Editor regression procedure

Use an editor session with no PIE running. Preserve the original automatic reload setting, and restore it afterwards. Do not save temporary test assets or configuration.

1. Select **On Script Save**. Add a temporary C# `UObject` class with a reflected, callable method returning `1`. Save and wait for a successful reload. Call the method and verify `1`.
2. Replace the return expression with an undefined symbol. Save. Verify that the complete diagnostic appears immediately in Output Log. Without clicking any notification, interact with the editor and call the existing method; it must still return `1`.
3. Leave the invalid source unchanged. Verify that the editor does not continuously compile it or stack failure notifications.
4. Fix the method to return `2`. Save. Verify that reload succeeds, the notification clears, and the method returns `2`.
5. Introduce another error. Switch to **On Editor Focus** and focus the editor. Leave it focused and verify that the unchanged error is not compiled on every tick.
6. Fix the source and focus the editor. Verify that the next automatic reload succeeds. Repeat with **On Script Save** to check recovery across multiple failure cycles.
7. Remove the temporary source, wait for reload, and restore the original setting.

Also check the file-update error path when diagnosing an unreadable source file: it should log the details and report a notification, without unloading the active assemblies or opening a modal error dialog.

## Validation scope

The patch was built and exercised in a Windows UE 5.8 Installed Build project using plugin baseline `2319b65`. Real editor probes verified callable versions before and after compile failures, immediate diagnostics, continued editor command responsiveness without dialog acknowledgement, idle failed-source behavior, repeated recovery, and removal of the temporary source. The same three-file patch applies to upstream `395a1cf` without conflicts; the complete latest upstream plugin was not substituted into that existing project for this validation.

The automated probe uses **On Script Save**. Focus-mode retry gating was checked in source; focused-window execution, notification link/dismiss interaction, and the unreadable-file failure path are additional manual checks not covered by that probe.
