# Smart Mode Logic Backup (Archived)

> [!NOTE]
> This file contains the complete 1900+ lines of logic for "Smart Mode" in the Macro Editor. 
> It was archived because it was temporarily disabled and hidden in the main app (which is currently hardcoded to Raw Mode).

## File Metadata
- **Original Path:** `PowerX_Keys_V2/ViewModels/MacroEditorViewModel.SmartView.cs`
- **Total Lines:** 1903
- **Archive Date:** 2026-07-07

---

```csharp
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Data;
using System.Windows.Threading;
using PowerX_Keys_V2.Models;
using PowerX_Keys_V2.Views;
using PowerX_Keys_V2.Services;
using PowerX_Keys_V2.Managers;

namespace PowerX_Keys_V2.ViewModels
{
    public partial class MacroEditorViewModel
    {
        private System.Threading.CancellationTokenSource _refreshCts;
        private System.Windows.Threading.DispatcherTimer _timelineOverlayTimer;
        private bool? _lastIsDelayHidden;

        private void ResetTimelineState()
        {
            Application.Current?.Dispatcher?.Invoke(() =>
            {
                _timelineOverlayTimer?.Stop();
                IsTimelineProcessing = false;
                OnPropertyChanged(nameof(IsTimelineEmpty));
            });
        }

        public void RefreshDisplaySteps(bool immediate = false)
        {
            if (CurrentMacro?.MacroSteps == null || SuspendDisplayRefresh) return;

            _refreshCts?.Cancel();
            _refreshCts?.Dispose();
            _refreshCts = new System.Threading.CancellationTokenSource();
            var token = _refreshCts.Token;

            _ = RunRefreshDisplayStepsAsync(token, immediate);
        }

        private async Task RunRefreshDisplayStepsAsync(System.Threading.CancellationToken token, bool immediate = false)
        {
            try
            {
                if (!immediate) await Task.Delay(50, token);

                if (CurrentMacro?.MacroSteps == null || SuspendDisplayRefresh)
                {
                    ResetTimelineState();
                    return;
                }

                List<StepSnapshot> snapshot = null;
                bool isSmart = IsSmartMode;
                bool enableModWrap = true; // Always-on: checkbox removed, Smart View always groups modifiers
                bool isDelayHiddenVal = IsDelayHidden;

                Application.Current?.Dispatcher?.Invoke(() =>
                {
                    // Smart delay: Don't show the loader immediately — start a 500ms timer.
                    // If processing finishes before the timer fires, user never sees the loader.
                    _timelineOverlayTimer?.Stop();
                    _timelineOverlayTimer = new System.Windows.Threading.DispatcherTimer();
                    _timelineOverlayTimer.Interval = TimeSpan.FromMilliseconds(500);
                    _timelineOverlayTimer.Tick += (s, args) =>
                    {
                        _timelineOverlayTimer.Stop();
                        IsTimelineProcessing = true;
                        OnPropertyChanged(nameof(IsTimelineEmpty));
                    };
                    _timelineOverlayTimer.Start();
                    
                    UpdateParentIdsRecursive(CurrentMacro.MacroSteps, null);
                    snapshot = CreateSnapshot(CurrentMacro.MacroSteps);
                });

                if (token.IsCancellationRequested)
                {
                    ResetTimelineState();
                    return;
                }

                var result = await Task.Run(() =>
                {
                    var target = new List<StepSnapshot>();
                    bool initialCapsLock = CurrentMacro?.InitialCapsLockOn ?? false;
                    PopulateDisplayStepsBackground(snapshot, target, isSmart, enableModWrap, 0, initialCapsLock);

                    int errorCount = 0;
                    void TraverseError(List<StepSnapshot> nodes)
                    {
                        if (nodes == null) return;
                        foreach (var snap in nodes)
                        {
                            if (!snap.IsDisabled && !snap.IsValid) errorCount++;
                            TraverseError(snap.ChildSteps);
                            TraverseError(snap.ChildStepsFalse);
                        }
                    }
                    TraverseError(snapshot);

                    return new { Target = target, ErrorCount = errorCount };
                }, token);

                if (token.IsCancellationRequested)
                {
                    ResetTimelineState();
                    return;
                }

                if (Application.Current?.Dispatcher == null) return;
                await Application.Current.Dispatcher.InvokeAsync(() =>
                {
                    if (token.IsCancellationRequested)
                    {
                        _timelineOverlayTimer?.Stop();
                        IsTimelineProcessing = false;
                        OnPropertyChanged(nameof(IsTimelineEmpty));
                        return;
                    }

                    var newSource = MapSnapshotToSteps(result.Target);

                    _internalSourceSteps.SyncTo(newSource, AreStepsEqual, SyncStepProperties);

                    ErrorCount = result.ErrorCount;

                    bool filterChanged = !_lastIsDelayHidden.HasValue || _lastIsDelayHidden.Value != isDelayHiddenVal;
                    _lastIsDelayHidden = isDelayHiddenVal;

                    if (DisplayMacroSteps == null)
                    {
                        DisplayMacroSteps = CollectionViewSource.GetDefaultView(_internalSourceSteps);
                        DisplayMacroSteps.Filter = (obj) =>
                        {
                            if (_lastIsDelayHidden == true && obj is MacroStep step && step.Type == MacroStepType.Delay) return false;
                            return true;
                        };
                    }
                    else if (filterChanged)
                    {
                        DisplayMacroSteps.Refresh();
                    }

                    _timelineOverlayTimer?.Stop();
                    IsTimelineProcessing = false;
                    IsEditorLoading = false;
                    OnPropertyChanged(nameof(IsTimelineEmpty));

                    if (_lastInsertedStep != null && Services.ConfigManager.Current.Settings.ShowInsertFeedback)
                    {
                        var stepToHighlight = _internalSourceSteps.FirstOrDefault(s => 
                            ReferenceEquals(s, _lastInsertedStep) || 
                            (s.VirtualSourceSteps != null && s.VirtualSourceSteps.Contains(_lastInsertedStep)));
                        
                        if (stepToHighlight != null)
                        {
                            _lastInsertedStep = null;
                            stepToHighlight.IsHighlightActive = true;
                            
                            // Request scroll in view
                            RequestScrollToStep(stepToHighlight);
                            
                            // Turn off highlight after 1.5s
                            _ = Task.Delay(1500).ContinueWith(t =>
                            {
                                Application.Current?.Dispatcher?.Invoke(() =>
                                {
                                    stepToHighlight.IsHighlightActive = false;
                                });
                            });
                        }
                    }
                    else
                    {
                        _lastInsertedStep = null;
                    }
                }, System.Windows.Threading.DispatcherPriority.Input);
            }
            catch (TaskCanceledException)
            {
                ResetTimelineState();
            }
            catch (Exception ex)
            {
                ResetTimelineState();
                System.Diagnostics.Debug.WriteLine($"Error in RefreshDisplaySteps: {ex.Message}");
            }
        }

        private void UpdateParentIdsRecursive(System.Collections.Generic.IEnumerable<Models.MacroStep> steps, Guid? parentId)
        {
            if (steps == null) return;
            foreach (var step in steps)
            {
                step.ParentId = parentId;
                if (step.ChildSteps != null && step.ChildSteps.Count > 0)
                {
                    UpdateParentIdsRecursive(step.ChildSteps, step.Id);
                }
                if (step.ChildStepsFalse != null && step.ChildStepsFalse.Count > 0)
                {
                    UpdateParentIdsRecursive(step.ChildStepsFalse, step.Id);
                }
            }
        }

        private void PopulateDisplaySteps(ObservableCollection<MacroStep> steps, List<MacroStep> target, int depth = 0)
        {
            var snapshot = CreateSnapshot(steps);
            var tempTarget = new List<StepSnapshot>();
            PopulateDisplayStepsBackground(snapshot, tempTarget, IsSmartMode, true, depth, false); // enableModWrap always-on
            var mapped = MapSnapshotToSteps(tempTarget);
            target.AddRange(mapped);
        }

        private class StepSnapshot
        {
            public MacroStep OriginalStep { get; set; }
            public MacroStepType Type { get; set; }
            public string Value { get; set; }
            public string KeyActionType { get; set; }
            public int? Duration { get; set; }
            public double? X { get; set; }
            public double? Y { get; set; }
            public double? EndX { get; set; }
            public double? EndY { get; set; }
            public string StepName { get; set; }
            public bool IsDisabled { get; set; }
            public bool IsValid { get; set; }
            public int? ScrollAmount { get; set; }
            public string ScrollPreset { get; set; }
            public int ClickCount { get; set; }
            public int Depth { get; set; }

            public List<StepSnapshot> ChildSteps { get; set; }
            public List<StepSnapshot> ChildStepsFalse { get; set; }
            public List<MacroStep> VirtualSourceSteps { get; set; }
            public bool IsRecordingStart { get; set; }
            public bool InitialCapsLock { get; set; }
        }

        private List<StepSnapshot> CreateSnapshot(IEnumerable<MacroStep> steps)
        {
            if (steps == null) return null;
            var list = new List<StepSnapshot>();
            foreach (var s in steps)
            {
                list.Add(new StepSnapshot
                {
                    OriginalStep = s,
                    Type = s.Type,
                    Value = s.Value,
                    KeyActionType = s.KeyActionType,
                    Duration = s.Duration,
                    X = s.X,
                    Y = s.Y,
                    EndX = s.EndX,
                    EndY = s.EndY,
                    StepName = s.StepName,
                    IsDisabled = s.IsDisabled,
                    IsValid = s.IsValid,
                    ScrollAmount = s.ScrollAmount,
                    ScrollPreset = s.ScrollPreset,
                    IsRecordingStart = s.IsRecordingStart,
                    InitialCapsLock = s.InitialCapsLock,
                    ChildSteps = CreateSnapshot(s.ChildSteps),
                    ChildStepsFalse = CreateSnapshot(s.ChildStepsFalse)
                });
            }
            return list;
        }

        private void PopulateDisplayStepsBackground(List<StepSnapshot> steps, List<StepSnapshot> target, bool isSmart, bool enableModWrap, int depth, bool initialCapsLockOn = false)
        {
            if (steps == null) return;
            for (int i = 0; i < steps.Count; i++)
            {
                var step = steps[i];

                // MODIFIER WRAP DETECTION (Smart View Only)
                if (isSmart && enableModWrap &&
                    step.Type == MacroStepType.Keyboard && step.KeyActionType == "Hold Down")
                {
                    string modVal = step.Value?.ToLower() ?? "";
                    bool isTopMod = IsModifierKey(modVal);
                    if (isTopMod)
                    {
                        // Smart Shift Detection: If Shift + 2+ typing keys, skip wrap → let text bundling handle it
                        bool skipShiftWrap = false;
                        if (modVal.Contains("shift"))
                        {
                            int typingKeyCount = 0;
                            bool hasNonTypingKey = false;
                            for (int peek = i + 1; peek < steps.Count; peek++)
                            {
                                var ps = steps[peek];
                                if (ps.Type == MacroStepType.Delay) continue;
                                if (ps.Type != MacroStepType.Keyboard) break;
                                string pv = ps.Value?.ToLower() ?? "";
                                if (pv.Contains("shift"))
                                {
                                    if (ps.KeyActionType == "Released Up" || ps.KeyActionType == "Release Up") break;
                                    continue; // Skip shift repeats
                                }
                                // Check for non-shift modifiers (Ctrl/Alt/Win) — if present, keep the wrap
                                bool isOtherMod = IsModifierKey(pv) && !pv.Contains("shift");
                                if (isOtherMod) { hasNonTypingKey = true; break; }
                                // Skip CapsLock — it's a toggle key, not a shortcut target
                                if (pv == "capslock") continue;
                                if (IsTypingKey(ps.Value) && ps.KeyActionType == "Hold Down")
                                    typingKeyCount++;
                                else if (!IsTypingKey(ps.Value) && ps.KeyActionType == "Hold Down")
                                { hasNonTypingKey = true; break; }
                            }
                            if (typingKeyCount >= 2 && !hasNonTypingKey)
                                skipShiftWrap = true;
                        }

                        if (!skipShiftWrap)
                        {
                        int j = i + 1;
                        bool validWrap = false;
                        List<StepSnapshot> innerSteps = new List<StepSnapshot>();
                        List<MacroStep> allSourceSteps = new List<MacroStep> { step.OriginalStep };
                        int targetCountBefore = target.Count;
                        MacroStep outerReleaseStep = null;

                        while (j < steps.Count)
                        {
                            var cur = steps[j];
                            if (cur.IsRecordingStart)
                            {
                                break;
                            }
                            if (cur.Type == MacroStepType.Delay)
                            {
                                innerSteps.Add(cur);
                                allSourceSteps.Add(cur.OriginalStep);
                                j++;
                                continue;
                            }
                            if (cur.Type == MacroStepType.Keyboard)
                            {
                                string curVal = cur.Value?.ToLower() ?? "";
                                bool isMatchingRelease = (curVal == modVal) &&
                                    (cur.KeyActionType == "Released Up" || cur.KeyActionType == "Release Up");
                                if (isMatchingRelease)
                                {
                                    allSourceSteps.Add(cur.OriginalStep);
                                    outerReleaseStep = cur.OriginalStep;
                                    validWrap = true;
                                    j++;
                                    break;
                                }
                                innerSteps.Add(cur);
                                allSourceSteps.Add(cur.OriginalStep);
                                j++;
                                continue;
                            }
                            // Tolerate non-keyboard/delay steps (e.g., mouse jiggles during recording)
                            innerSteps.Add(cur);
                            allSourceSteps.Add(cur.OriginalStep);
                            j++;
                            continue;
                        }

                        // FIX: If no matching release found (recording stopped while modifier held),
                        // still allow wrap if inner steps contain actual key presses
                        if (!validWrap && innerSteps.Count > 0 &&
                            innerSteps.Any(s => s.Type == MacroStepType.Keyboard && s.KeyActionType == "Hold Down" &&
                                !IsModifierKey(s.Value?.ToLower() ?? "")))
                        {
                            validWrap = true;
                        }

                        if (validWrap && innerSteps.Count > 0)
                        {
                            var currentChunk = new List<MacroStep>();
                            var activeModifierSteps = new List<MacroStep>();
                            var keysPressedInChunk = new List<string>();
                            List<string> chunkBaseModifiers = new List<string> { modVal };
                            StepSnapshot lastVirtualStep = null;
                            bool broken = false;

                            int k = 0;
                            while (k < innerSteps.Count)
                            {
                                var inner = innerSteps[k];
                                if (inner.Type == MacroStepType.Delay)
                                {
                                    int delayMs = inner.Duration ?? inner.OriginalStep?.Duration ?? 0;
                                    bool isSignificantDelay = delayMs >= 800;

                                    if (currentChunk.Count > 0 && (isSignificantDelay || keysPressedInChunk.Count > 0))
                                    {
                                        if (keysPressedInChunk.Count > 0)
                                        {
                                            List<string> baseModParts = new List<string>();
                                            foreach (var m in chunkBaseModifiers)
                                            {
                                                string label = NormalizeModifierLabel(m);
                                                if (label != null && !baseModParts.Contains(label)) baseModParts.Add(label);
                                            }

                                            string shortcutName = string.Join(" + ", baseModParts);
                                            foreach (var pk in keysPressedInChunk) shortcutName += (string.IsNullOrEmpty(shortcutName) ? "" : " + ") + pk.ToUpper();

                                            var virtualShortcutStep = new StepSnapshot
                                            {
                                                Type = MacroStepType.Keyboard,
                                                Value = shortcutName,
                                                KeyActionType = "Press",
                                                StepName = $"Shortcut: {shortcutName}",
                                                Depth = depth,
                                                VirtualSourceSteps = new List<MacroStep>(currentChunk)
                                            };
                                            target.Add(virtualShortcutStep);
                                            lastVirtualStep = virtualShortcutStep;
                                        }

                                        currentChunk.Clear();
                                        keysPressedInChunk.Clear();
                                    }

                                    // Only emit significant delays (>= 800ms) to display; consume small delays into source steps
                                    if (isSignificantDelay)
                                    {
                                        target.Add(new StepSnapshot
                                        {
                                            OriginalStep = inner.OriginalStep,
                                            Type = MacroStepType.Delay,
                                            Duration = delayMs,
                                            Depth = depth
                                        });
                                    }
                                    else
                                    {
                                        // Absorb small delay into the last virtual step or allSourceSteps
                                        if (lastVirtualStep != null)
                                        {
                                            if (lastVirtualStep.VirtualSourceSteps == null)
                                                lastVirtualStep.VirtualSourceSteps = new List<MacroStep>();
                                            lastVirtualStep.VirtualSourceSteps.Add(inner.OriginalStep);
                                        }
                                    }
                                    k++;
                                }
                                else if (inner.Type == MacroStepType.Keyboard)
                                {
                                    string val = inner.Value?.ToLower() ?? "";
                                    bool isMod = IsModifierKey(val);

                                    if (isMod)
                                    {
                                        if (inner.KeyActionType == "Hold Down")
                                        {
                                            // FIX Case 1 & 2: Flush pending keys BEFORE adding new modifier
                                            if (keysPressedInChunk.Count > 0)
                                            {
                                                List<string> flushModParts = new List<string>();
                                                foreach (var m in chunkBaseModifiers)
                                                {
                                                    string label = NormalizeModifierLabel(m);
                                                    if (label != null && !flushModParts.Contains(label)) flushModParts.Add(label);
                                                }
                                                string flushName = string.Join(" + ", flushModParts);
                                                foreach (var pk in keysPressedInChunk) flushName += (string.IsNullOrEmpty(flushName) ? "" : " + ") + pk.ToUpper();

                                                var flushStep = new StepSnapshot
                                                {
                                                    Type = MacroStepType.Keyboard,
                                                    Value = flushName,
                                                    KeyActionType = "Press",
                                                    StepName = $"Shortcut: {flushName}",
                                                    Depth = depth,
                                                    VirtualSourceSteps = new List<MacroStep>(currentChunk)
                                                };
                                                target.Add(flushStep);
                                                lastVirtualStep = flushStep;
                                                currentChunk.Clear();
                                                // Re-add active modifier steps so they're shared with the next combo block
                                                currentChunk.AddRange(activeModifierSteps);
                                                keysPressedInChunk.Clear();
                                            }

                                            if (!chunkBaseModifiers.Contains(val)) chunkBaseModifiers.Add(val);
                                            currentChunk.Add(inner.OriginalStep);
                                            activeModifierSteps.Add(inner.OriginalStep);
                                            k++;
                                        }
                                        else if (inner.KeyActionType == "Released Up" || inner.KeyActionType == "Release Up")
                                        {
                                            currentChunk.Add(inner.OriginalStep);
                                            chunkBaseModifiers.Remove(val);
                                            activeModifierSteps.RemoveAll(x => x.Value?.ToLower() == val);
                                            k++;
                                        }
                                        else
                                        {
                                            broken = true;
                                            break;
                                        }
                                    }
                                    else
                                    {
                                        if (inner.KeyActionType == "Hold Down")
                                        {
                                            // If we already have keys pressed in this chunk and we're seeing a NEW key press,
                                            // flush the current chunk as a separate shortcut (e.g., Ctrl+C done, now Ctrl+V starts)
                                            if (keysPressedInChunk.Count > 0)
                                            {
                                                List<string> baseModParts2 = new List<string>();
                                                foreach (var m in chunkBaseModifiers)
                                                {
                                                    string label = NormalizeModifierLabel(m);
                                                    if (label != null && !baseModParts2.Contains(label)) baseModParts2.Add(label);
                                                }
                                                string shortcutName2 = string.Join(" + ", baseModParts2);
                                                foreach (var pk in keysPressedInChunk) shortcutName2 += (string.IsNullOrEmpty(shortcutName2) ? "" : " + ") + pk.ToUpper();

                                                var virtualShortcutStep2 = new StepSnapshot
                                                {
                                                    Type = MacroStepType.Keyboard,
                                                    Value = shortcutName2,
                                                    KeyActionType = "Press",
                                                    StepName = $"Shortcut: {shortcutName2}",
                                                    Depth = depth,
                                                    VirtualSourceSteps = new List<MacroStep>(currentChunk)
                                                };
                                                target.Add(virtualShortcutStep2);
                                                lastVirtualStep = virtualShortcutStep2;

                                                currentChunk.Clear();
                                                // Re-add active modifier steps so they're shared with the next combo block
                                                currentChunk.AddRange(activeModifierSteps);
                                                keysPressedInChunk.Clear();
                                            }

                                            keysPressedInChunk.Add(inner.Value);
                                            currentChunk.Add(inner.OriginalStep);
                                            k++;
                                        }
                                        else if (inner.KeyActionType == "Released Up" || inner.KeyActionType == "Release Up")
                                        {
                                            currentChunk.Add(inner.OriginalStep);
                                            k++;
                                        }
                                        else
                                        {
                                            broken = true;
                                            break;
                                        }
                                    }
                                }
                                else
                                {
                                    // Tolerate non-keyboard/delay inner steps (absorb into current chunk source)
                                    currentChunk.Add(inner.OriginalStep);
                                    k++;
                                }
                            }

                            if (!broken && lastVirtualStep != null)
                            {
                                if (currentChunk.Count > 0)
                                {
                                    if (keysPressedInChunk.Count > 0)
                                    {
                                        List<string> baseModParts = new List<string>();
                                        foreach (var m in chunkBaseModifiers)
                                        {
                                            string label = NormalizeModifierLabel(m);
                                            if (label != null && !baseModParts.Contains(label)) baseModParts.Add(label);
                                        }

                                        string shortcutName = string.Join(" + ", baseModParts);
                                        foreach (var pk in keysPressedInChunk) shortcutName += (string.IsNullOrEmpty(shortcutName) ? "" : " + ") + pk.ToUpper();

                                        var virtualShortcutStep = new StepSnapshot
                                        {
                                            Type = MacroStepType.Keyboard,
                                            Value = shortcutName,
                                            KeyActionType = "Press",
                                            StepName = $"Shortcut: {shortcutName}",
                                            Depth = depth,
                                            VirtualSourceSteps = new List<MacroStep>(currentChunk)
                                        };
                                        target.Add(virtualShortcutStep);
                                    }
                                    else
                                    {
                                        if (lastVirtualStep.VirtualSourceSteps == null)
                                            lastVirtualStep.VirtualSourceSteps = new List<MacroStep>();
                                        foreach (var s in currentChunk) lastVirtualStep.VirtualSourceSteps.Add(s);
                                    }
                                }
                                // FIX: Include outer modifier Hold/Release in all virtual blocks for clean deletion
                                for (int idx = targetCountBefore; idx < target.Count; idx++)
                                {
                                    var t = target[idx];
                                    if (t.VirtualSourceSteps != null && t.Type == MacroStepType.Keyboard && t.KeyActionType == "Press")
                                    {
                                        t.VirtualSourceSteps.Insert(0, step.OriginalStep);
                                        if (outerReleaseStep != null)
                                            t.VirtualSourceSteps.Add(outerReleaseStep);
                                    }
                                }
                                i = j - 1;
                                continue;
                            }
                            else if (!broken && activeModifierSteps.Count == 0 && keysPressedInChunk.Count > 0)
                            {
                                List<string> baseModParts = new List<string>();
                                foreach (var m in chunkBaseModifiers)
                                {
                                    string label = NormalizeModifierLabel(m);
                                    if (label != null && !baseModParts.Contains(label)) baseModParts.Add(label);
                                }

                                string shortcutName = string.Join(" + ", baseModParts);
                                foreach (var pk in keysPressedInChunk) shortcutName += " + " + pk.ToUpper();

                                var virtualShortcutStep = new StepSnapshot
                                {
                                    Type = MacroStepType.Keyboard,
                                    Value = shortcutName,
                                    KeyActionType = "Press",
                                    StepName = $"Shortcut: {shortcutName}",
                                    Depth = depth,
                                    VirtualSourceSteps = new List<MacroStep>(currentChunk)
                                };
                                target.Add(virtualShortcutStep);
                                // FIX: Include outer modifier Hold/Release in virtual block for clean deletion
                                for (int idx = targetCountBefore; idx < target.Count; idx++)
                                {
                                    var t = target[idx];
                                    if (t.VirtualSourceSteps != null && t.Type == MacroStepType.Keyboard && t.KeyActionType == "Press")
                                    {
                                        t.VirtualSourceSteps.Insert(0, step.OriginalStep);
                                        if (outerReleaseStep != null)
                                            t.VirtualSourceSteps.Add(outerReleaseStep);
                                    }
                                }
                                i = j - 1;
                                continue;
                            }
                        }
                        } // end if (!skipShiftWrap)
                    }
                }

                // SIMPLE KEY PRESS BUNDLER (Smart View Only)
                // If a keyboard Hold Down is directly followed by Released Up for the SAME key
                // (with only small delays in between, no other keys), collapse into a single "Press".
                // NOTE: Typing keys (letters, numbers) are excluded — they go to the text bundler below.
                if (isSmart &&
                    step.Type == MacroStepType.Keyboard &&
                    step.KeyActionType == "Hold Down" &&
                    !IsTypingKey(step.Value))
                {
                    int j = i + 1;
                    var sourceSteps = new List<MacroStep> { step.OriginalStep };
                    bool foundMatchingRelease = false;

                    while (j < steps.Count)
                    {
                        var next = steps[j];
                        if (next.IsRecordingStart) break;

                        if (next.Type == MacroStepType.Delay && (next.Duration ?? 0) < 800)
                        {
                            sourceSteps.Add(next.OriginalStep);
                            j++;
                            continue;
                        }

                        // Found a keyboard event — check if it's the matching release
                        if (next.Type == MacroStepType.Keyboard &&
                            string.Equals(next.Value, step.Value, StringComparison.OrdinalIgnoreCase) &&
                            (next.KeyActionType == "Released Up" || next.KeyActionType == "Release Up"))
                        {
                            sourceSteps.Add(next.OriginalStep);
                            foundMatchingRelease = true;
                            j++;
                        }
                        break; // Any other step breaks the pair
                    }

                    if (foundMatchingRelease)
                    {
                        string displayName = step.Value ?? "";
                        // Clean up common internal key names for display
                        string lower = displayName.ToLower();
                        string modLabel = NormalizeModifierLabel(lower);
                        if (modLabel != null)
                            displayName = char.ToUpper(modLabel[0]) + modLabel.Substring(1).ToLower();

                        var virtualPressStep = new StepSnapshot
                        {
                            Type            = MacroStepType.Keyboard,
                            Value           = displayName,
                            KeyActionType   = "Press",
                            StepName        = $"Key Press: {displayName}",
                            Depth           = depth,
                            VirtualSourceSteps = sourceSteps
                        };
                        target.Add(virtualPressStep);
                        i = j - 1;
                        continue;
                    }
                }

                // VISUAL TEXT BUNDLING (Smart View Only)
                // Skip if a non-shift modifier (Ctrl/Alt/Win) is held — keys are part of a shortcut, not typing
                if (isSmart && step.Type == MacroStepType.Keyboard && step.KeyActionType == "Hold Down" &&
                    IsTypingKey(step.Value))
                {
                    bool modifierHeld = false;
                    for (int k = i - 1; k >= 0; k--)
                    {
                        var prev = steps[k];
                        if (prev.Type != MacroStepType.Keyboard) continue;
                        string pv = prev.Value?.ToLower() ?? "";
                        bool isMod = IsModifierKey(pv) && !pv.Contains("shift");
                        if (isMod)
                        {
                            if (prev.KeyActionType == "Hold Down") { modifierHeld = true; break; }
                            if (prev.KeyActionType == "Released Up" || prev.KeyActionType == "Release Up") break;
                        }
                    }
                    if (!modifierHeld)
                {
                    int j = i;
                    string bundledText = "";
                    List<MacroStep> sourceSteps = new List<MacroStep>();
                    bool isShiftDown = false;
                    bool isCapsLockOn = GetCapsLockStateAt(steps, i, initialCapsLockOn);

                    // Look backward to see if Shift is already held down
                    for (int k = i - 1; k >= 0; k--)
                    {
                        var prev = steps[k];
                        if (prev.Type == MacroStepType.Keyboard && (prev.Value?.ToLower().Contains("shift") == true))
                        {
                            if (prev.KeyActionType == "Hold Down") isShiftDown = true;
                            break;
                        }
                    }

                    // FIX Case 3: Consume leaked Shift Hold + delay from target
                    // When skipShiftWrap=true, the Shift Hold falls through as a raw step.
                    // If isShiftDown, remove the orphaned Shift Hold and any trailing delay from target.
                    if (isShiftDown)
                    {
                        bool foundShift = false;
                        while (target.Count > 0 && !foundShift)
                        {
                            var last = target[target.Count - 1];
                            bool isLeakedDelay = (last.Type == MacroStepType.Delay) ||
                                (last.OriginalStep != null && last.OriginalStep.Type == MacroStepType.Delay);
                            bool isLeakedShiftHold = last.Type == MacroStepType.Keyboard &&
                                (last.Value?.ToLower().Contains("shift") == true) &&
                                (last.KeyActionType == "Hold Down" || last.OriginalStep?.KeyActionType == "Hold Down");

                            if (isLeakedDelay)
                            {
                                if (last.OriginalStep != null) sourceSteps.Insert(0, last.OriginalStep);
                                target.RemoveAt(target.Count - 1);
                            }
                            else if (isLeakedShiftHold)
                            {
                                if (last.OriginalStep != null) sourceSteps.Insert(0, last.OriginalStep);
                                target.RemoveAt(target.Count - 1);
                                foundShift = true;
                            }
                            else break;
                        }
                    }

                    while (j < steps.Count)
                    {
                        var current = steps[j];
                        if (j > i && current.IsRecordingStart)
                        {
                            break;
                        }
                        if (current.Type == MacroStepType.Delay)
                        {
                            if ((current.Duration ?? 0) < 800)
                            {
                                sourceSteps.Add(current.OriginalStep);
                                j++;
                                continue;
                            }
                            else break;
                        }
                        else if (current.Type == MacroStepType.Keyboard)
                        {
                            string val = current.Value;
                            if (val != null && val.ToLower().Contains("shift"))
                            {
                                sourceSteps.Add(current.OriginalStep);
                                if (current.KeyActionType == "Hold Down") isShiftDown = true;
                                else if (current.KeyActionType == "Released Up" || current.KeyActionType == "Release Up") isShiftDown = false;
                                j++;
                            }
                            // FIX Case 4: Handle CapsLock toggles during text bundling
                            else if (val != null && val.Equals("capslock", StringComparison.OrdinalIgnoreCase))
                            {
                                sourceSteps.Add(current.OriginalStep);
                                if (current.KeyActionType == "Hold Down") isCapsLockOn = !isCapsLockOn;
                                j++;
                            }
                            else if (IsTypingKey(val))
                            {
                                sourceSteps.Add(current.OriginalStep);
                                if (current.KeyActionType == "Hold Down")
                                {
                                    string c = GetTypingChar(val);
                                    // FIX Case 4: Shift XOR CapsLock for uppercase (standard keyboard behavior)
                                    if ((isShiftDown ^ isCapsLockOn) && c.Length == 1) c = c.ToUpper();
                                    bundledText += c;
                                }
                                j++;
                            }
                            else if (val != null && val.Equals("backspace", StringComparison.OrdinalIgnoreCase))
                            {
                                sourceSteps.Add(current.OriginalStep);
                                if (current.KeyActionType == "Hold Down" && bundledText.Length > 0)
                                    bundledText = bundledText.Substring(0, bundledText.Length - 1);
                                j++;
                            }
                            else
                            {
                                break;
                            }
                        }
                        else break;
                    }

                    if (bundledText.Length > 0 && sourceSteps.Count >= 2)
                    {
                        var virtualTextStep = new StepSnapshot
                        {
                            Type = MacroStepType.Text,
                            Value = bundledText,
                            StepName = $"Type '{bundledText}'",
                            Depth = depth,
                            VirtualSourceSteps = sourceSteps
                        };
                        target.Add(virtualTextStep);
                        i = j - 1;
                        continue;
                    }
                    } // end if (!modifierHeld)
                }

                // VISUAL MOUSE BUNDLING (Smart View Only)
                if (isSmart && step.Type == MacroStepType.MouseClick)
                {
                    if (step.Value == "Hold Down" || step.Value == "Right Down")
                    {
                        bool isRight = step.Value == "Right Down";
                        string expectedRelease = isRight ? "Right Up" : "Released Up";

                        int j = i + 1;
                        List<MacroStep> sourceSteps = new List<MacroStep> { step.OriginalStep };
                        bool foundRelease = false;
                        bool hasTrace = false;
                        StepSnapshot releaseStep = null;

                        while (j < steps.Count)
                        {
                            var current = steps[j];
                            if (current.IsRecordingStart)
                            {
                                break;
                            }
                            sourceSteps.Add(current.OriginalStep);

                            if (current.Type == MacroStepType.MouseClick && current.Value == expectedRelease)
                            {
                                foundRelease = true;
                                releaseStep = current;
                                j++;
                                break;
                            }
                            else if (current.Type == MacroStepType.Delay)
                            {
                                j++;
                            }
                            else if (current.Type == MacroStepType.MouseTrace)
                            {
                                hasTrace = true;
                                j++;
                            }
                            else
                            {
                                break;
                            }
                        }

                        if (foundRelease)
                        {
                            if (hasTrace || step.X != releaseStep.X || step.Y != releaseStep.Y)
                            {
                                var virtualDragStep = new StepSnapshot
                                {
                                    Type = MacroStepType.MouseClick,
                                    Value = isRight ? "Right Drag and Drop" : "Drag and Drop",
                                    X = step.X,
                                    Y = step.Y,
                                    EndX = releaseStep.X ?? sourceSteps.LastOrDefault(s => s.Type == MacroStepType.MouseTrace)?.X ?? step.X,
                                    EndY = releaseStep.Y ?? sourceSteps.LastOrDefault(s => s.Type == MacroStepType.MouseTrace)?.Y ?? step.Y,
                                    StepName = isRight ? "Right Drag and Drop" : "Drag and Drop",
                                    Depth = depth,
                                    VirtualSourceSteps = sourceSteps
                                };
                                target.Add(virtualDragStep);
                                i = j - 1;
                                continue;
                            }
                            else
                            {
                                int totalClicks = 1; // We already have the first click (hold+release)
                                int k = j;

                                // Keep scanning for additional click pairs at approximately the same position
                                while (true)
                                {
                                    List<MacroStep> lookaheadSteps = new List<MacroStep>();
                                    int kStart = k;
                                    bool foundNextHold = false;

                                    // Look for the next Hold Down (skip small delays)
                                    while (k < steps.Count)
                                    {
                                        var next = steps[k];
                                        lookaheadSteps.Add(next.OriginalStep);

                                        if (next.Type == MacroStepType.Delay)
                                        {
                                            if ((next.Duration ?? 0) > 500) break;
                                            k++;
                                        }
                                        else if (next.Type == MacroStepType.MouseClick && next.Value == step.Value)
                                        {
                                            // Check if click is in approximately the same area (±20px tolerance)
                                            bool sameArea = Math.Abs((next.X ?? 0) - (step.X ?? 0)) <= 20 &&
                                            Math.Abs((next.Y ?? 0) - (step.Y ?? 0)) <= 20;
                                            if (sameArea)
                                            {
                                                foundNextHold = true;
                                                k++;
                                            }
                                            break;
                                        }
                                        else
                                        {
                                            break;
                                        }
                                    }

                                    if (!foundNextHold)
                                    {
                                        k = kStart; // Reset — don't consume these steps
                                        break;
                                    }

                                    // Look for the matching Release
                                    bool foundNextRelease = false;
                                    while (k < steps.Count)
                                    {
                                        var next = steps[k];
                                        if (!lookaheadSteps.Contains(next.OriginalStep))
                                            lookaheadSteps.Add(next.OriginalStep);

                                        if (next.Type == MacroStepType.MouseClick && next.Value == expectedRelease)
                                        {
                                            foundNextRelease = true;
                                            k++;
                                            break;
                                        }
                                        else if (next.Type == MacroStepType.Delay)
                                        {
                                            k++;
                                        }
                                        else
                                        {
                                            break;
                                        }
                                    }

                                    if (foundNextRelease)
                                    {
                                        totalClicks++;
                                        sourceSteps.AddRange(lookaheadSteps);
                                        j = k;
                                    }
                                    else
                                    {
                                        k = kStart; // Reset — incomplete click pair
                                        break;
                                    }
                                }

                                string clickName;
                                int clickCount = 1;
                                if (totalClicks >= 3)
                                {
                                    clickName = "Multiple Clicks";
                                    clickCount = totalClicks;
                                }
                                else if (totalClicks == 2)
                                {
                                    clickName = isRight ? "Right Double Click" : "Left Double Click";
                                }
                                else
                                {
                                    clickName = isRight ? "Right Click" : "Left Click";
                                }

                                var virtualClickStep = new StepSnapshot
                                {
                                    Type = MacroStepType.MouseClick,
                                    Value = clickName,
                                    X = step.X,
                                    Y = step.Y,
                                    StepName = clickName,
                                    Depth = depth,
                                    ClickCount = clickCount,
                                    VirtualSourceSteps = sourceSteps
                                };
                                target.Add(virtualClickStep);
                                i = j - 1;
                                continue;
                            }
                        }
                    }
                    else if (step.Value != null && step.Value.StartsWith("Scroll "))
                    {
                        string scrollDir = step.Value;
                        int scrollCount = 1;
                        int j = i + 1;
                        List<MacroStep> sourceSteps = new List<MacroStep> { step.OriginalStep };

                        while (j < steps.Count)
                        {
                            var current = steps[j];
                            if (current.IsRecordingStart)
                            {
                                break;
                            }
                            if (current.Type == MacroStepType.Delay)
                            {
                                if ((current.Duration ?? 0) > 500) break;
                                sourceSteps.Add(current.OriginalStep);
                                j++;
                            }
                            else if (current.Type == MacroStepType.MouseClick && current.Value == scrollDir)
                            {
                                scrollCount++;
                                sourceSteps.Add(current.OriginalStep);
                                j++;
                            }
                            else
                            {
                                break;
                            }
                        }

                        if (scrollCount > 1)
                        {
                            var virtualScrollStep = new StepSnapshot
                            {
                                Type = MacroStepType.MouseClick,
                                Value = scrollDir,
                                StepName = scrollDir,
                                ScrollAmount = scrollCount,
                                ScrollPreset = (scrollCount == 1 || scrollCount == 2 || scrollCount == 3 || scrollCount == 5 || scrollCount == 10) ? scrollCount.ToString() : "Custom",
                                Depth = depth,
                                VirtualSourceSteps = sourceSteps
                            };
                            target.Add(virtualScrollStep);
                            i = j - 1;
                            continue;
                        }
                    }
                }

                step.Depth = depth;
                target.Add(step);
            }

            // Post-processing: Merge adjacent visual delays that are now next to each other in target (Smart View Only)
            if (isSmart)
            {
                // Helper to detect if a snapshot is a delay (handles the Type=0 bug where Type wasn't set)
                bool IsDelaySnapshot(StepSnapshot s)
                {
                    if (s.Type == MacroStepType.Delay) return true;
                    if (s.OriginalStep != null && s.OriginalStep.Type == MacroStepType.Delay) return true;
                    return false;
                }
                int GetDelayMs(StepSnapshot s)
                {
                    if (s.Duration.HasValue) return s.Duration.Value;
                    if (s.OriginalStep?.Duration != null) return s.OriginalStep.Duration.Value;
                    return 0;
                }

                // Pass 1: Merge adjacent delays
                for (int i = 0; i < target.Count - 1; i++)
                {
                    if (IsDelaySnapshot(target[i]) && IsDelaySnapshot(target[i + 1]))
                    {
                        var d1 = target[i];
                        var d2 = target[i + 1];
                        if (d1.OriginalStep?.IsManuallyAdded == true || d2.OriginalStep?.IsManuallyAdded == true)
                        {
                            continue;
                        }

                        var sourceDelays = new List<MacroStep>();
                        if (d1.OriginalStep != null) sourceDelays.Add(d1.OriginalStep);
                        else if (d1.VirtualSourceSteps != null) sourceDelays.AddRange(d1.VirtualSourceSteps);

                        if (d2.OriginalStep != null) sourceDelays.Add(d2.OriginalStep);
                        else if (d2.VirtualSourceSteps != null) sourceDelays.AddRange(d2.VirtualSourceSteps);

                        var mergedDelay = new StepSnapshot
                        {
                            Type = MacroStepType.Delay,
                            Duration = GetDelayMs(d1) + GetDelayMs(d2),
                            Depth = d1.Depth,
                            VirtualSourceSteps = sourceDelays
                        };

                        target[i] = mergedDelay;
                        target.RemoveAt(i + 1);
                        i--;
                    }
                }

                // Pass 2: Absorb small delays (< 1s) between consecutive keyboard shortcut steps
                // This turns [Shortcut A] → [Wait 400ms] → [Shortcut B] into just [Shortcut A] → [Shortcut B]
                for (int i = 0; i < target.Count - 2; i++)
                {
                    bool isVirtualShortcutA = target[i].Type == MacroStepType.Keyboard && target[i].KeyActionType == "Press";
                    bool middleIsSmallDelay = IsDelaySnapshot(target[i + 1]) && GetDelayMs(target[i + 1]) < 1000;
                    bool isVirtualShortcutB = target[i + 2].Type == MacroStepType.Keyboard && target[i + 2].KeyActionType == "Press";

                    if (isVirtualShortcutA && middleIsSmallDelay && isVirtualShortcutB)
                    {
                        if (target[i].OriginalStep?.IsManuallyAdded == true || 
                            target[i + 1].OriginalStep?.IsManuallyAdded == true || 
                            target[i + 2].OriginalStep?.IsManuallyAdded == true)
                        {
                            continue;
                        }

                        // Absorb the delay into shortcut A's VirtualSourceSteps
                        var delay = target[i + 1];
                        if (target[i].VirtualSourceSteps == null) target[i].VirtualSourceSteps = new List<MacroStep>();
                        if (delay.OriginalStep != null) target[i].VirtualSourceSteps.Add(delay.OriginalStep);
                        else if (delay.VirtualSourceSteps != null) target[i].VirtualSourceSteps.AddRange(delay.VirtualSourceSteps);

                        target.RemoveAt(i + 1);
                        i--; // Re-check in case there are more shortcuts in sequence
                    }
                }

                // Pass 2b: Absorb small leading delays (< 1s) that appear right before a keyboard shortcut
                // This handles the case after combo deletion: [Wait 247ms] → [Ctrl+V] → just [Ctrl+V]
                for (int i = 0; i < target.Count - 1; i++)
                {
                    bool isSmallDelay = IsDelaySnapshot(target[i]) && GetDelayMs(target[i]) < 1000;
                    bool nextIsShortcut = target[i + 1].Type == MacroStepType.Keyboard && target[i + 1].KeyActionType == "Press";
                    bool prevIsShortcut = i > 0 && target[i - 1].Type == MacroStepType.Keyboard && target[i - 1].KeyActionType == "Press";

                    if (isSmallDelay && nextIsShortcut && !prevIsShortcut)
                    {
                        if (target[i].OriginalStep?.IsManuallyAdded == true || 
                            target[i + 1].OriginalStep?.IsManuallyAdded == true)
                        {
                            continue;
                        }

                        // Absorb the delay into the following shortcut's VirtualSourceSteps
                        var delay = target[i];
                        if (target[i + 1].VirtualSourceSteps == null) target[i + 1].VirtualSourceSteps = new List<MacroStep>();
                        if (delay.OriginalStep != null) target[i + 1].VirtualSourceSteps.Insert(0, delay.OriginalStep);
                        else if (delay.VirtualSourceSteps != null) target[i + 1].VirtualSourceSteps.InsertRange(0, delay.VirtualSourceSteps);

                        target.RemoveAt(i);
                        i--;
                    }
                }

                // Pass 2c: Absorb small trailing delays (< 1s) that appear right after a keyboard shortcut
                // This handles: [Ctrl+C] → [Wait 247ms] → just [Ctrl+C] (after deleting Ctrl+V)
                for (int i = 0; i < target.Count - 1; i++)
                {
                    bool isShortcut = target[i].Type == MacroStepType.Keyboard && target[i].KeyActionType == "Press";
                    bool nextIsSmallDelay = IsDelaySnapshot(target[i + 1]) && GetDelayMs(target[i + 1]) < 1000;
                    bool afterDelayIsShortcut = (i + 2 < target.Count) && target[i + 2].Type == MacroStepType.Keyboard && target[i + 2].KeyActionType == "Press";

                    if (isShortcut && nextIsSmallDelay && !afterDelayIsShortcut)
                    {
                        if (target[i].OriginalStep?.IsManuallyAdded == true || 
                            target[i + 1].OriginalStep?.IsManuallyAdded == true)
                        {
                            continue;
                        }

                        // Absorb the trailing delay into the shortcut's VirtualSourceSteps
                        var delay = target[i + 1];
                        if (target[i].VirtualSourceSteps == null) target[i].VirtualSourceSteps = new List<MacroStep>();
                        if (delay.OriginalStep != null) target[i].VirtualSourceSteps.Add(delay.OriginalStep);
                        else if (delay.VirtualSourceSteps != null) target[i].VirtualSourceSteps.AddRange(delay.VirtualSourceSteps);

                        target.RemoveAt(i + 1);
                        i--;
                    }
                }

                // Pass 3: Absorb small delays (< 1s) between MousePath and MouseClick
                // [MousePath] → [Wait 300ms] → [Click] becomes [MousePath] → [Click]
                for (int i = 0; i < target.Count - 2; i++)
                {
                    bool isMousePath = target[i].Type == MacroStepType.MouseTrace ||
                        (target[i].OriginalStep != null && target[i].OriginalStep.Type == MacroStepType.MouseTrace);
                    bool middleIsSmallDelay = IsDelaySnapshot(target[i + 1]) && GetDelayMs(target[i + 1]) < 1000;
                    bool isMouseClick = target[i + 2].Type == MacroStepType.MouseClick;

                    if (isMousePath && middleIsSmallDelay && isMouseClick)
                    {
                        if (target[i + 1].OriginalStep?.IsManuallyAdded == true)
                        {
                            continue;
                        }

                        // Absorb the delay into the mouse path's VirtualSourceSteps
                        var delay = target[i + 1];
                        if (target[i].VirtualSourceSteps == null)
                            target[i].VirtualSourceSteps = new List<MacroStep>();
                        if (delay.OriginalStep != null) target[i].VirtualSourceSteps.Add(delay.OriginalStep);
                        else if (delay.VirtualSourceSteps != null) target[i].VirtualSourceSteps.AddRange(delay.VirtualSourceSteps);

                        target.RemoveAt(i + 1);
                        i--;
                    }
                }

                // Pass 4: Merge [MousePath] + [Drag and Drop] into a single Drag block
                // The path was the cursor movement leading into the drag, so they're one action
                for (int i = 0; i < target.Count - 1; i++)
                {
                    bool isMousePath = target[i].Type == MacroStepType.MouseTrace ||
                        (target[i].OriginalStep != null && target[i].OriginalStep.Type == MacroStepType.MouseTrace);
                    bool isDragAndDrop = target[i + 1].Type == MacroStepType.MouseClick &&
                        (target[i + 1].Value == "Drag and Drop" || target[i + 1].Value == "Right Drag and Drop");

                    if (isMousePath && isDragAndDrop)
                    {
                        // Absorb the mouse path into the drag step's VirtualSourceSteps
                        var pathStep = target[i];
                        var dragStep = target[i + 1];

                        if (dragStep.VirtualSourceSteps == null)
                            dragStep.VirtualSourceSteps = new List<MacroStep>();

                        // Insert path's source steps at the beginning of drag's source steps
                        if (pathStep.OriginalStep != null)
                            dragStep.VirtualSourceSteps.Insert(0, pathStep.OriginalStep);
                        if (pathStep.VirtualSourceSteps != null)
                        {
                            for (int p = pathStep.VirtualSourceSteps.Count - 1; p >= 0; p--)
                                dragStep.VirtualSourceSteps.Insert(0, pathStep.VirtualSourceSteps[p]);
                        }

                        target.RemoveAt(i);
                        i--;
                    }
                }
            }
        }

        private bool GetCapsLockStateAt(List<StepSnapshot> steps, int index, bool defaultInitialCapsLock)
        {
            int startIdx = 0;
            bool baseState = defaultInitialCapsLock;

            for (int k = index; k >= 0; k--)
            {
                if (steps[k].IsRecordingStart)
                {
                    startIdx = k;
                    baseState = steps[k].InitialCapsLock;
                    break;
                }
            }

            bool capsOn = baseState;
            for (int k = startIdx; k < index; k++)
            {
                var step = steps[k];
                if (step.Type == MacroStepType.Keyboard &&
                    step.Value?.Equals("capslock", StringComparison.OrdinalIgnoreCase) == true &&
                    step.KeyActionType == "Hold Down")
                {
                    capsOn = !capsOn;
                }
            }
            return capsOn;
        }

        private List<MacroStep> MapSnapshotToSteps(List<StepSnapshot> snapshots)
        {
            var list = new List<MacroStep>();
            foreach (var snap in snapshots)
            {
                if (snap.OriginalStep != null)
                {
                    snap.OriginalStep.Depth = snap.Depth;
                    snap.OriginalStep.VirtualSourceSteps = snap.VirtualSourceSteps;
                    list.Add(snap.OriginalStep);
                }
                else
                {
                    var virtualStep = new MacroStep
                    {
                        Type = snap.Type,
                        Value = snap.Value,
                        KeyActionType = snap.KeyActionType,
                        StepName = snap.StepName,
                        Depth = snap.Depth,
                        Duration = snap.Duration,
                        ScrollAmount = snap.ScrollAmount,
                        ScrollPreset = snap.ScrollPreset,
                        ClickCount = snap.ClickCount > 0 ? snap.ClickCount : 1,
                        EndX = snap.EndX,
                        EndY = snap.EndY,
                        X = snap.X,
                        Y = snap.Y,
                        VirtualSourceSteps = snap.VirtualSourceSteps
                    };

                    if (virtualStep.Type == MacroStepType.Text)
                    {
                        virtualStep.PropertyChanged += (s, e) => {
                            if (e.PropertyName == "Value") 
                                HandleVirtualTextChange(virtualStep);
                        };
                    }
                    else if (virtualStep.Type == MacroStepType.Delay)
                    {
                        virtualStep.PropertyChanged += (s, e) => {
                            if (e.PropertyName == "Duration") 
                                HandleVirtualDelayChange(virtualStep);
                        };
                    }
                    else if (virtualStep.Type == MacroStepType.MouseClick && virtualStep.VirtualSourceSteps != null)
                    {
                        // F20 fix: sync coordinate edits in SmartView back to the underlying raw steps
                        virtualStep.PropertyChanged += (s, e) =>
                        {
                            if (e.PropertyName == "X" || e.PropertyName == "Y" ||
                                e.PropertyName == "EndX" || e.PropertyName == "EndY")
                                HandleVirtualMouseCoordChange(virtualStep);
                        };
                    }

                    list.Add(virtualStep);
                }
            }
            return list;
        }

        private bool AreStepsEqual(MacroStep a, MacroStep b)
        {
            if (ReferenceEquals(a, b)) return true;
            if (a == null || b == null) return false;

            bool aIsVirtual = a.VirtualSourceSteps != null;
            bool bIsVirtual = b.VirtualSourceSteps != null;
            if (aIsVirtual != bIsVirtual) return false;

            if (aIsVirtual && bIsVirtual)
            {
                if (a.Type != b.Type) return false;
                if (a.Value != b.Value) return false;
                if (a.StepName != b.StepName) return false;
                if (a.ClickCount != b.ClickCount) return false;
                if (a.VirtualSourceSteps.Count != b.VirtualSourceSteps.Count) return false;

                for (int i = 0; i < a.VirtualSourceSteps.Count; i++)
                {
                    if (!ReferenceEquals(a.VirtualSourceSteps[i], b.VirtualSourceSteps[i])) return false;
                }

                return true;
            }

            return a.Id == b.Id;
        }

        private void SyncStepProperties(MacroStep target, MacroStep source)
        {
            if (target.Value != source.Value) target.Value = source.Value;
            if (target.StepName != source.StepName) target.StepName = source.StepName;
            if (target.Depth != source.Depth) target.Depth = source.Depth;
            if (target.ClickCount != source.ClickCount) target.ClickCount = source.ClickCount;
            target.VirtualSourceSteps = source.VirtualSourceSteps; // Always sync (reference identity changes)
            if (target.Type == MacroStepType.Delay && target.Duration != source.Duration)
            {
                target.Duration = source.Duration;
            }
        }

        public async Task SimulateLiveBuildAsync(List<MacroStep> stepsToBuild, bool append = false, int delayMs = 800)
        {
            if (CurrentMacro == null || stepsToBuild == null) return;
            if (CurrentMacro.MacroSteps == null) CurrentMacro.MacroSteps = new ObservableCollection<MacroStep>();

            Application.Current.Dispatcher.Invoke(() => IsLiveBuildInProgress = true);

            if (!append)
            {
                CurrentMacro.MacroSteps.Clear();
                ForceRefreshDisplaySteps();
                await Task.Delay(500);
            }

            foreach (var step in stepsToBuild)
            {
                Application.Current.Dispatcher.Invoke(() => { CurrentMacro?.MacroSteps?.Add(step); });
                await Task.Delay(delayMs);
            }

            await Task.Delay(1500);
            Application.Current.Dispatcher.Invoke(() =>
            {
                if (CurrentMacro != null) IsDirty = true;
                IsLiveBuildInProgress = false;
            });
        }
        public MacroStep InsertStep(MacroStepType type, int index)
        {
            if (CurrentMacro?.MacroSteps == null) return null;
            UndoRedoManager.PushState(CurrentMacro.MacroSteps);
            MacroStep macroStep = CreateNewStep(type);
            macroStep.IsManuallyAdded = true;

            DeselectAllSteps(CurrentMacro.MacroSteps);
            macroStep.IsSelected = true;

            if (index < 0) index = 0;
            if (index > CurrentMacro.MacroSteps.Count) index = CurrentMacro.MacroSteps.Count;

            CurrentMacro.MacroSteps.Insert(index, macroStep);

            // Instant load: add placeholder version to the UI collection instantly
            InstantInsertStepToUI(macroStep, CurrentMacro.MacroSteps, index);

            IsDirty = true;
            _lastInsertedStep = macroStep;
            ForceRefreshDisplaySteps();
            return macroStep;
        }
        private string GetNextBlockName(MacroStepType type, string baseName)
        {
            int maxNum = 0;
            if (CurrentMacro != null && CurrentMacro.MacroSteps != null)
            {
                CheckForMaxNum(CurrentMacro.MacroSteps, type, baseName, ref maxNum);
            }
            return $"{baseName} {maxNum + 1}";
        }

        private void CheckForMaxNum(IEnumerable<MacroStep> steps, MacroStepType type, string baseName, ref int maxNum)
        {
            if (steps == null) return;
            foreach (var step in steps)
            {
                if (step.Type == type && step.Value != null && step.Value.StartsWith(baseName + " "))
                {
                    string s = step.Value.Substring(baseName.Length + 1);
                    if (int.TryParse(s, out var result) && result > maxNum)
                    {
                        maxNum = result;
                    }
                }
                if (step.ChildSteps != null) CheckForMaxNum(step.ChildSteps, type, baseName, ref maxNum);
                if (step.ChildStepsFalse != null) CheckForMaxNum(step.ChildStepsFalse, type, baseName, ref maxNum);
            }
        }
        private string GetNextStepName(MacroStepType type, string baseName)
        {
            int maxNum = 0;
            if (CurrentMacro != null && CurrentMacro.MacroSteps != null)
            {
                CheckForMaxStepName(CurrentMacro.MacroSteps, type, baseName, ref maxNum);
            }
            return $"{baseName} {maxNum + 1}";
        }

        private void CheckForMaxStepName(IEnumerable<MacroStep> steps, MacroStepType type, string baseName, ref int maxNum)
        {
            if (steps == null) return;
            foreach (var step in steps)
            {
                if (step.Type == type && step.StepName != null && step.StepName.StartsWith(baseName + " "))
                {
                    string s = step.StepName.Substring(baseName.Length + 1);
                    if (int.TryParse(s, out var result) && result > maxNum)
                    {
                        maxNum = result;
                    }
                }
                if (step.ChildSteps != null) CheckForMaxStepName(step.ChildSteps, type, baseName, ref maxNum);
                if (step.ChildStepsFalse != null) CheckForMaxStepName(step.ChildStepsFalse, type, baseName, ref maxNum);
            }
        }
        private void HandleVirtualTextChange(MacroStep virtualStep)
        {
            if (virtualStep.VirtualSourceSteps == null || virtualStep.VirtualSourceSteps.Count == 0) return;
            
            // Push undo state before destructive modification so the user can Ctrl+Z
            UndoRedoManager.PushState(CurrentMacro.MacroSteps);

            // 1. Find the parent collection and the range of original steps
            var parentCollection = FindParentCollection(CurrentMacro.MacroSteps, virtualStep.VirtualSourceSteps[0]);
            if (parentCollection == null) return;

            int firstIndex = parentCollection.IndexOf(virtualStep.VirtualSourceSteps[0]);
            int lastIndex = parentCollection.IndexOf(virtualStep.VirtualSourceSteps.Last());
            
            if (firstIndex < 0 || lastIndex < 0) return;

            // 2. Extract old chunks to preserve original recorded steps (Bug #7 fix)
            var oldChunks = ExtractCharacterChunks(virtualStep.VirtualSourceSteps);

            // 3. Generate new raw steps from the new text, reusing old chunks where possible
            var newSteps = RegenerateKeyboardStepsFromText(virtualStep.Value, oldChunks);

            // 3. Swap the old steps for new ones
            int countToRemove = lastIndex - firstIndex + 1;
            
            // Perform the swap on the main collection
            for (int k = 0; k < countToRemove; k++) parentCollection.RemoveAt(firstIndex);
            for (int k = 0; k < newSteps.Count; k++) parentCollection.Insert(firstIndex + k, newSteps[k]);

            IsDirty = true;
            // Trigger a UI refresh to reflect the changes (and re-bundle if necessary)
            Application.Current.Dispatcher.InvokeAsync(() => RefreshDisplaySteps());
        }

        private void HandleVirtualDelayChange(MacroStep virtualStep)
        {
            if (virtualStep.VirtualSourceSteps == null || virtualStep.VirtualSourceSteps.Count == 0) return;

            // Push undo state before destructive modification so the user can Ctrl+Z
            UndoRedoManager.PushState(CurrentMacro.MacroSteps);

            var firstRaw = virtualStep.VirtualSourceSteps[0];
            var parentCollection = FindParentCollection(CurrentMacro.MacroSteps, firstRaw);
            if (parentCollection == null) return;

            int newDuration = virtualStep.Duration ?? 0;
            
            // Set the first raw delay's duration to the new value
            firstRaw.Duration = newDuration;

            // Remove subsequent raw delays from the parent collection
            for (int i = 1; i < virtualStep.VirtualSourceSteps.Count; i++)
            {
                var rawStep = virtualStep.VirtualSourceSteps[i];
                parentCollection.Remove(rawStep);
            }

            // Clear the virtual source list to only contain the first raw step now
            virtualStep.VirtualSourceSteps.Clear();
            virtualStep.VirtualSourceSteps.Add(firstRaw);

            IsDirty = true;
            
            // Trigger a UI refresh to reflect the changes
            Application.Current.Dispatcher.InvokeAsync(() => RefreshDisplaySteps());
        }

        /// <summary>
        /// F20 fix: Syncs X/Y/EndX/EndY edits on a virtual mouse step back to the underlying raw MacroSteps.
        /// For clicks: the first source step gets the updated X/Y.
        /// For drags:  the first source step gets X/Y (hold) and the last source step gets EndX/EndY (release).
        /// </summary>
        private void HandleVirtualMouseCoordChange(MacroStep virtualStep)
        {
            if (virtualStep.VirtualSourceSteps == null || virtualStep.VirtualSourceSteps.Count == 0) return;

            // Push undo state before destructive modification
            UndoRedoManager.PushState(CurrentMacro.MacroSteps);

            // The first source step is always the mouse-down / hold step — update its origin coordinates.
            var holdStep = virtualStep.VirtualSourceSteps[0];
            holdStep.X = virtualStep.X;
            holdStep.Y = virtualStep.Y;

            // For drag-and-drop the last source step is the release step — update its destination coordinates.
            if (virtualStep.VirtualSourceSteps.Count > 1)
            {
                var releaseStep = virtualStep.VirtualSourceSteps[virtualStep.VirtualSourceSteps.Count - 1];
                releaseStep.X = virtualStep.EndX;
                releaseStep.Y = virtualStep.EndY;
            }

            IsDirty = true;
            OnPropertyChanged(nameof(IsSaveReady));
            Application.Current.Dispatcher.InvokeAsync(() => RefreshDisplaySteps());
        }

        private List<List<MacroStep>> ExtractCharacterChunks(List<MacroStep> sourceSteps)
        {
            var chunks = new List<List<MacroStep>>();
            List<MacroStep> currentChunk = null;
            
            foreach (var step in sourceSteps)
            {
                if (step.Type == MacroStepType.Keyboard && step.KeyActionType == "Hold Down" && IsTypingKey(step.Value))
                {
                    if (currentChunk != null) chunks.Add(currentChunk);
                    currentChunk = new List<MacroStep>();
                }
                if (currentChunk != null)
                {
                    currentChunk.Add(step);
                }
            }
            if (currentChunk != null) chunks.Add(currentChunk);
            return chunks;
        }

        private List<MacroStep> RegenerateKeyboardStepsFromText(string text, List<List<MacroStep>> oldChunks)
        {
            var steps = new List<MacroStep>();
            if (string.IsNullOrEmpty(text)) return steps;

            var unusedChunks = oldChunks != null ? new List<List<MacroStep>>(oldChunks) : new List<List<MacroStep>>();

            for (int k = 0; k < text.Length; k++)
            {
                char c = text[k];
                string keyVal = c.ToString();
                if (c == ' ') keyVal = "Space";
                
                // Try to find and reuse an existing chunk for this character
                List<MacroStep> matchingChunk = null;
                for (int i = 0; i < unusedChunks.Count; i++)
                {
                    var chunk = unusedChunks[i];
                    var downStep = chunk.FirstOrDefault(s => s.Type == MacroStepType.Keyboard && s.KeyActionType == "Hold Down");
                    if (downStep != null && downStep.Value.Equals(keyVal, StringComparison.OrdinalIgnoreCase))
                    {
                        matchingChunk = chunk;
                        unusedChunks.RemoveAt(i);
                        break;
                    }
                }

                if (matchingChunk != null)
                {
                    // If this is the last character, strip out any trailing synthetic delay
                    if (k == text.Length - 1)
                    {
                        var lastChunkStep = matchingChunk.LastOrDefault();
                        if (lastChunkStep != null && lastChunkStep.Type == MacroStepType.Delay)
                        {
                            matchingChunk.RemoveAt(matchingChunk.Count - 1);
                        }
                    }
                    steps.AddRange(matchingChunk);
                }
                else
                {
                    // Add Down
                    steps.Add(new MacroStep { 
                        Type = MacroStepType.Keyboard, 
                        Value = keyVal, 
                        KeyActionType = "Hold Down", 
                        IsRegenerated = true,
                        IsSynthetic = true 
                    });
                    
                    // Add Small Delay (e.g. 10ms)
                    steps.Add(new MacroStep { 
                        Type = MacroStepType.Delay, 
                        Duration = 10, 
                        IsRegenerated = true,
                        IsSynthetic = true 
                    });
                    
                    // Add Up
                    steps.Add(new MacroStep { 
                        Type = MacroStepType.Keyboard, 
                        Value = keyVal, 
                        KeyActionType = "Released Up", 
                        IsRegenerated = true,
                        IsSynthetic = true 
                    });

                    // Add Delay between keys (ONLY if it's not the last character)
                    if (k < text.Length - 1)
                    {
                        steps.Add(new MacroStep { 
                            Type = MacroStepType.Delay, 
                            Duration = 20, 
                            IsRegenerated = true,
                            IsSynthetic = true 
                        });
                    }
                }
            }
            return steps;
        }

        /// <summary>
        /// Determines if a key name represents a standard typing key (letter, digit, space, punctuation).
        /// Used by the Smart View text bundling engine.
        /// </summary>
        private bool IsTypingKey(string keyValue)
        {
            if (string.IsNullOrEmpty(keyValue)) return false;
            if (keyValue.Length == 1 && (char.IsLetterOrDigit(keyValue[0]) || char.IsPunctuation(keyValue[0]) || char.IsSymbol(keyValue[0]))) return true;
            if (keyValue.Equals("space", StringComparison.OrdinalIgnoreCase)) return true;
            if (keyValue.Equals("tab", StringComparison.OrdinalIgnoreCase)) return true;
            return false;
        }

        /// <summary>
        /// Converts a key name to the character it represents for display in bundled text.
        /// </summary>
        private string GetTypingChar(string keyValue)
        {
            if (keyValue.Equals("space", StringComparison.OrdinalIgnoreCase)) return " ";
            if (keyValue.Equals("tab", StringComparison.OrdinalIgnoreCase)) return "\t";
            if (keyValue.Length == 1) return keyValue.ToLower();
            return keyValue;
        }

        public MacroStep FindParentStepByChildCollection(ObservableCollection<MacroStep> steps, ObservableCollection<MacroStep> targetCollection)
        {
            if (targetCollection == null || steps == null) return null;
            foreach (var step in steps)
            {
                if (ReferenceEquals(step.ChildSteps, targetCollection) || ReferenceEquals(step.ChildStepsFalse, targetCollection)) return step;
                if (step.ChildSteps != null)
                {
                    var result = FindParentStepByChildCollection(step.ChildSteps, targetCollection);
                    if (result != null) return result;
                }
                if (step.ChildStepsFalse != null)
                {
                    var result = FindParentStepByChildCollection(step.ChildStepsFalse, targetCollection);
                    if (result != null) return result;
                }
            }
            return null;
        }

        public void InstantInsertStepToUI(MacroStep newStep, ObservableCollection<MacroStep> parentCollection, int indexInParent)
        {
            if (newStep == null || parentCollection == null) return;

            int visualInsertIndex = -1;
            int depth = 0;

            // 1. Calculate depth
            if (parentCollection != CurrentMacro?.MacroSteps)
            {
                var parentStep = FindParentStep(CurrentMacro?.MacroSteps, parentCollection.FirstOrDefault() ?? newStep);
                if (parentStep == null)
                {
                    parentStep = FindParentStepByChildCollection(CurrentMacro?.MacroSteps, parentCollection);
                }
                if (parentStep != null)
                {
                    depth = parentStep.Depth + 1;
                }
            }
            newStep.Depth = depth;

            // 2. Find visual insert position
            if (indexInParent == 0)
            {
                if (parentCollection == CurrentMacro?.MacroSteps)
                {
                    visualInsertIndex = 0;
                }
                else
                {
                    var parentStep = FindParentStepByChildCollection(CurrentMacro?.MacroSteps, parentCollection);
                    if (parentStep != null)
                    {
                        int idx = -1;
                        for (int i = 0; i < _internalSourceSteps.Count; i++)
                        {
                            var item = _internalSourceSteps[i];
                            if (ReferenceEquals(item, parentStep) || 
                                (item.VirtualSourceSteps != null && item.VirtualSourceSteps.Contains(parentStep)))
                            {
                                idx = i;
                                break;
                            }
                        }
                        if (idx >= 0)
                        {
                            visualInsertIndex = idx + 1;
                        }
                    }
                }
            }
            else
            {
                if (indexInParent - 1 >= 0 && indexInParent - 1 < parentCollection.Count)
                {
                    var prevStep = parentCollection[indexInParent - 1];
                    int idx = -1;
                    for (int i = 0; i < _internalSourceSteps.Count; i++)
                    {
                        var item = _internalSourceSteps[i];
                        if (ReferenceEquals(item, prevStep) || 
                            (item.VirtualSourceSteps != null && item.VirtualSourceSteps.Contains(prevStep)))
                        {
                            idx = i;
                            break;
                        }
                    }
                    if (idx >= 0)
                    {
                        visualInsertIndex = idx + 1;
                    }
                }
            }

            // Insert into the flat UI list
            if (visualInsertIndex >= 0 && visualInsertIndex <= _internalSourceSteps.Count)
            {
                _internalSourceSteps.Insert(visualInsertIndex, newStep);
            }
            else
            {
                _internalSourceSteps.Add(newStep);
            }
        }

        /// <summary>
        /// Instantly removes a step from the visible UI list so the deletion feels
        /// immediate. The async RefreshDisplaySteps() call that follows will reconcile
        /// the full display, but the user sees the card vanish right away.
        /// </summary>
        public void InstantRemoveStepFromUI(MacroStep step)
        {
            if (step == null || _internalSourceSteps == null) return;

            // Direct match — step is in the display list as-is
            if (_internalSourceSteps.Remove(step)) return;

            // Virtual step match — a display card wraps this step in VirtualSourceSteps
            for (int i = _internalSourceSteps.Count - 1; i >= 0; i--)
            {
                var displayStep = _internalSourceSteps[i];
                if (displayStep.VirtualSourceSteps != null && displayStep.VirtualSourceSteps.Contains(step))
                {
                    _internalSourceSteps.RemoveAt(i);
                    return;
                }
            }
        }

        private static string NormalizeModifierLabel(string rawMod)
        {
            switch (rawMod)
            {
                case "lcontrol": case "rcontrol": return "CTRL";
                case "lshift": case "rshift": return "SHIFT";
                case "lalt": case "ralt": return "ALT";
                case "lwin": case "rwin": return "WIN";
                default: return null;
            }
        }

        private static bool IsModifierKey(string val)
        {
            return val == "lcontrol" || val == "rcontrol" || val == "lshift" || val == "rshift" ||
                   val == "lalt" || val == "ralt" || val == "lwin" || val == "rwin";
        }
    }
}
```
